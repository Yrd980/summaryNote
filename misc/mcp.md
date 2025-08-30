Awesome—here’s a **concrete mini-project** . It shows **MCP server + client**, with **Roots** (scoping), **Elicitation** (ask user mid-flow), and **Sampling** (server asks client to run an LLM call).

---

# Project layout

```
mcp-demo/
├─ client/
│  ├─ host.ts            # MCP client host/orchestrator
│  ├─ transports.ts      # stdio/http stream helpers
│  └─ ui.ts              # user prompts + approval gates
└─ server.release-notes/
   ├─ server.ts          # MCP server (stdio)
   ├─ tools.ts           # Tools (generateReleaseNotes, readFile, writeFile)
   └─ fs-sandbox.ts      # Roots enforcement & path guards
```

---

# Shared types (mental model)

```ts
// ---- wire-level-ish concepts (simplified) ----
type JsonRpcReq = { id: string; method: string; params?: any };
type JsonRpcRes = { id: string; result?: any; error?: { code: number; message: string } };

// Capabilities server advertises after initialize
type Capabilities = {
  tools: Array<{
    name: string;
    description: string;
    inputSchema: any;
    outputSchema: any;
    supportsElicitation?: boolean;
    maySample?: boolean; // server might ask for nested LLM calls
  }>;
  resources?: Array<{ uriRoot: string }>; // e.g., "workspace://repoA"
};

// Stream events (server → client) during a tool invocation
type StreamEvent =
  | { type: "log"; level: "info" | "warn" | "error"; message: string }
  | { type: "partial"; data: any }
  | { type: "elicitation"; prompt: string; schema: any; nonce: string }
  | { type: "sampling-request"; nonce: string; prompt: string; system?: string; maxTokens?: number }
  | { type: "done" }
  | { type: "error"; message: string };
```

---

# Server (stdio) — `server.release-notes/server.ts`

```ts
// Pseudocode server over stdio
import { listTools, invokeTool } from "./tools";

let ROOTS = new Set<string>(); // e.g., "workspace://repoA"
let PROTOCOL_VERSION = "2025-08-01";

async function main() {
  for await (const msg of readJsonRpcFromSTDIN()) {
    if (msg.method === "initialize") {
      const reqRoots: string[] = msg.params?.roots ?? [];
      ROOTS = new Set(reqRoots); // <-- ROOTS: client-scoped access
      writeJsonRpc({
        id: msg.id,
        result: {
          protocolVersion: PROTOCOL_VERSION,
          capabilities: {
            tools: listTools(),           // advertise tools
            resources: [...ROOTS].map(r => ({ uriRoot: r })),
          } satisfies Capabilities
        }
      });
    }

    else if (msg.method === "invokeTool") {
      const { name, args } = msg.params;
      try {
        // Streamed execution
        for await (const ev of invokeTool(name, args, { ROOTS }, serverSideHooks)) {
          writeStreamEvent(ev); // emits log/partial/elicitation/sampling-request/done
        }
        writeJsonRpc({ id: msg.id, result: { ok: true } });
      } catch (e) {
        writeStreamEvent({ type: "error", message: String(e) });
        writeJsonRpc({ id: msg.id, error: { code: 500, message: String(e) } });
      }
    }

    // Client answers elicitation/sampling:
    else if (msg.method === "elicitationResponse") {
      serverSideHooks.resumeElicitation(msg.params.nonce, msg.params.answer);
    } else if (msg.method === "samplingResponse") {
      serverSideHooks.resumeSampling(msg.params.nonce, msg.params.output);
    }
  }
}

// Simple coordination hooks for elicitation/sampling awaits
const serverSideHooks = (() => {
  const elicitWaiters = new Map<string, (answer: any) => void>();
  const sampleWaiters = new Map<string, (text: string) => void>();

  return {
    requestElicitation(prompt: string, schema: any): Promise<any> {
      const nonce = cryptoRandomId();
      writeStreamEvent({ type: "elicitation", prompt, schema, nonce });
      return new Promise(res => elicitWaiters.set(nonce, res));
    },
    resumeElicitation(nonce: string, answer: any) {
      elicitWaiters.get(nonce)?.(answer);
      elicitWaiters.delete(nonce);
    },
    requestSampling(prompt: string, opts?: { system?: string; maxTokens?: number }): Promise<string> {
      const nonce = cryptoRandomId();
      writeStreamEvent({ type: "sampling-request", nonce, prompt, ...opts });
      return new Promise(res => sampleWaiters.set(nonce, res));
    },
    resumeSampling(nonce: string, output: string) {
      sampleWaiters.get(nonce)?.(output);
      sampleWaiters.delete(nonce);
    }
  };
})();

main();
```

---

# Server tools — `server.release-notes/tools.ts`

```ts
import { ensureWithinRoots, listCommits, readFile, writeFile } from "./fs-sandbox";

export function listTools() {
  return [
    {
      name: "generateReleaseNotes",
      description: "Summarize commits between two tags/refs and draft release notes.",
      inputSchema: { type: "object", properties: {
        repo: { type: "string", format: "uri" },       // e.g., "workspace://repoA"
        fromRef: { type: "string" },
        toRef: { type: "string" },
        style: { type: "string", enum: ["concise", "detailed"], default: "concise" },
        outputFile: { type: "string" } // must live under the repo Root
      }, required: ["repo"] },
      outputSchema: { type: "object", properties: { path: { type: "string" } } },
      supportsElicitation: true,
      maySample: true
    }
  ];
}

export async function* invokeTool(name: string, args: any, ctx: { ROOTS: Set<string> }, hooks: any) {
  if (name !== "generateReleaseNotes") throw new Error("Unknown tool");
  let { repo, fromRef, toRef, style = "concise", outputFile } = args;

  ensureWithinRoots(ctx.ROOTS, repo, outputFile); // ---- Roots enforcement

  // If missing args, elicit just-in-time
  if (!fromRef || !toRef) {
    const answer = await hooks.requestElicitation(
      "Select a range to summarize",
      { type: "object", properties: {
        mode: { type: "string", enum: ["sinceTag", "range"] },
        sinceTag: { type: "string" },
        fromRef: { type: "string" }, toRef: { type: "string" }
      }}
    );
    if (answer.mode === "sinceTag") {
      fromRef = answer.sinceTag;
      toRef = "HEAD";
    } else {
      fromRef = answer.fromRef;
      toRef = answer.toRef;
    }
  }

  yield { type: "log", level: "info", message: `Collecting commits ${fromRef}..${toRef}` };
  const commits = await listCommits(repo, fromRef, toRef); // returns [{hash, message}, ...]

  // Ask client to run the LLM (Sampling)
  const prompt = `Summarize these commits as ${style} release notes:\n` +
                 commits.map(c => `- ${c.hash.slice(0,7)} ${c.message}`).join("\n");
  const notes = await hooks.requestSampling(prompt, { system: "You are a release-notes assistant." });

  yield { type: "partial", data: { preview: notes.slice(0, 200) } };

  // Write result under allowed root
  const target = outputFile ?? `${repo}/RELEASE_NOTES.md`;
  await writeFile(target, notes);
  yield { type: "log", level: "info", message: `Wrote ${target}` };
  yield { type: "done" };
}
```

---

# Roots guard (server) — `server.release-notes/fs-sandbox.ts`

```ts
export function ensureWithinRoots(roots: Set<string>, repoUri: string, maybeOut?: string) {
  if (!roots.has(prefixOf(repoUri))) throw new Error(`Repo not permitted by Roots: ${repoUri}`);
  if (maybeOut && !maybeOut.startsWith(prefixOf(repoUri))) {
    throw new Error(`Output path must be under repo root: ${maybeOut}`);
  }
}
function prefixOf(uri: string) {
  // e.g., "workspace://repoA/path/file" -> "workspace://repoA"
  const m = uri.match(/^[a-z]+:\/\/[^/]+/i);
  return m ? m[0] : uri;
}

// Mocked helpers
export async function listCommits(repo: string, fromRef: string, toRef: string) { /* ... */ return [] as any[]; }
export async function readFile(uri: string) { /* ... */ return ""; }
export async function writeFile(uri: string, content: string) { /* ... */ }
```

---

# Client host (connect, discover, invoke, **drain stream**) — `client/host.ts`

```ts
import { connectStdio } from "./transports";
import { askUser, confirmSampling } from "./ui";

async function main() {
  const proc = connectStdio("node", ["server.release-notes/server.js"]);

  // Initialize with Roots (scopes)
  const roots = ["workspace://repoA"];
  const initRes = await rpc(proc, { method: "initialize", params: { roots } });
  const caps = initRes.result.capabilities;

  // Find tool
  const tool = caps.tools.find(t => t.name === "generateReleaseNotes");
  if (!tool) throw new Error("Server missing tool");

  // Kick off tool invocation
  const inv = await rpc(proc, {
    method: "invokeTool",
    params: { name: tool.name, args: { repo: "workspace://repoA", style: "detailed" } }
  });

  // ---- STREAM CONSUMPTION IS CRITICAL ----
  // Drain all events until 'done' or 'error'
  for await (const ev of proc.streamEvents()) {
    if (ev.type === "log") console.log(`[${ev.level}]`, ev.message);
    if (ev.type === "partial") console.log("Preview:", ev.data.preview);

    // Elicitation UX
    if (ev.type === "elicitation") {
      const answer = await askUser(ev.prompt, ev.schema);
      await rpc(proc, { method: "elicitationResponse", params: { nonce: ev.nonce, answer } });
    }

    // Sampling (nested LLM) UX + approval gate
    if (ev.type === "sampling-request") {
      const ok = await confirmSampling(ev.prompt);
      if (!ok) {
        await rpc(proc, { method: "samplingResponse", params: { nonce: ev.nonce, output: "Sampling denied by user." } });
        continue;
      }
      // Call your local/remote LLM here
      const llmText = await runLLM(ev.prompt, { system: ev.system, maxTokens: ev.maxTokens });
      await rpc(proc, { method: "samplingResponse", params: { nonce: ev.nonce, output: llmText } });
    }

    if (ev.type === "done") break;
    if (ev.type === "error") throw new Error(ev.message);
  }

  // Even if we ignored content, we'd still need to DRAIN:
  // async for (_ of proc.streamEvents()) { /* pass */ }  // ensures clean close
}

main().catch(console.error);

// ---- util RPC ----
async function rpc(proc: any, req: Omit<JsonRpcReq,"id">) : Promise<JsonRpcRes> {
  const id = cryptoRandomId();
  proc.send({ id, ...req });
  return await proc.waitForResponse(id);
}

// Pretend LLM
async function runLLM(prompt: string, opts?: any): Promise<string> {
  // Call your model; return text
  return `# Release Notes\n\n- Example item derived from:\n${prompt.substring(0,120)}...`;
}
```

---

# Transport helpers — `client/transports.ts`

```ts
export function connectStdio(cmd: string, args: string[]) {
  const child = spawn(cmd, args, { stdio: ["pipe","pipe","pipe"] });

  const pending = new Map<string,(res:any)=>void>();
  const emitter = makeAsyncEventEmitter<StreamEvent>();

  // read lines → parse JSON → route responses or stream events
  child.stdout.on("data", (buf) => {
    for (const line of splitJsonLines(buf)) {
      const msg = JSON.parse(line);
      if ("result" in msg || "error" in msg) {
        pending.get(msg.id)?.(msg);
        pending.delete(msg.id);
      } else if (msg.type) {
        emitter.emit(msg as StreamEvent);
      }
    }
  });

  return {
    send: (obj: any) => child.stdin.write(JSON.stringify(obj) + "\n"),
    waitForResponse: (id: string) => new Promise(res => pending.set(id,res)),
    async *streamEvents() { for await (const ev of emitter) yield ev; }
  };
}
```

---

# Client UI stubs — `client/ui.ts`

```ts
export async function askUser(prompt: string, schema: any) {
  // Show a small form; validate to schema; return object
  return { mode: "sinceTag", sinceTag: "v2.0.0" };
}
export async function confirmSampling(prompt: string) {
  // Show confirmation modal with the prompt preview
  return true; // user approves
}
```

---

# Why this demonstrates the three concepts cleanly

* **Roots**: client passes `roots = ["workspace://repoA"]` during `initialize`; server enforces with `ensureWithinRoots(...)`.
* **Elicitation**: server lacks `fromRef/toRef`, so it emits an **elicitation** event; client shows a UI and replies with `elicitationResponse`.
* **Sampling**: server needs summarization; it emits **sampling-request**; client optionally asks the user, runs the LLM, and replies with `samplingResponse`.

And on the client side we **drain the stream** (looping over `proc.streamEvents()`), which is essential for completing the invocation and releasing resources.

### Sampling (optional)

**What it is:** A server can *ask the client to run an LLM call* **during** a tool/resource operation (a “nested” completion) and use the result before finishing the request. ([Model Context Protocol][1], [Model Context Protocol][2], [Daily Dose of Data Science][3])

**Why it exists:** Some tools need on-the-spot reasoning (e.g., summarize fetched docs, draft a commit message, classify results). Instead of baking a model into the server, the server **requests** the client’s model to “sample” text and returns with that answer inlined.

**Trust/UX note:** Specs recommend a human-in-the-loop gate; clients can show a consent UI (“This tool wants to call the model—allow?”). Not all clients support it yet. ([Model Context Protocol][1], [Model Context Protocol][2])

**Analogy:** The tool is a chef; halfway through cooking it asks the diner (client+LLM), “Taste this—should I add salt?” The diner replies; the chef finishes the dish.

---

### Roots

**What it is:** A way for the **client** to declare “these are the directories/URIs you’re allowed to touch,” so the **server** understands its scope/boundaries. Roots can also update dynamically. ([Model Context Protocol][4], [Model Context Protocol][5])

**Why it exists:** Clear sandboxing & discoverability. A filesystem tool can limit itself to, say, `workspace://repoA/` and ignore everything else. It also helps the client surface relevant resources in UI (only show tools where they “apply”). ([Model Context Protocol][4])

**Analogy:** Think of Roots as a **map legend** the client hands the server: “You may operate inside these fenced gardens—nowhere else.”

---

### Elicitation

**What it is:** Lets a **server** pause and **request structured user input mid-tool** (missing parameter, clarification, extra context) instead of cramming everything into the initial call. ([Model Context Protocol][6], [Daily Dose of Data Science][7], [FastMCP][8])

**Why it exists:** Interactive, resilient workflows. Tools can ask, “Which branch should I use?” or “Date range?” and then continue. Clients should guard against sensitive asks and present clear UI. ([Model Context Protocol][6])

**Analogy:** A wizard form that pops one precise question at the exact moment it’s needed—no bloated prompts up front.

---

### How these guide discovery/UX together

* **Roots → discovery & scoping:** The client can show only the files/areas a server may act on, and the server tailors its offers to those areas. ([Model Context Protocol][4])
* **Elicitation → smooth input UX:** Instead of failing for missing args, the tool converses to get exactly what it needs. ([Model Context Protocol][6])
* **Sampling → richer results:** Tools can incorporate LLM reasoning *on demand* (with consent), producing cleaner final outputs without the client having to orchestrate every micro-step. ([Model Context Protocol][1])

---

### Tiny concrete flow (end-to-end)

1. Client declares Roots: `workspace://repoA`, `vault://notes`. ([Model Context Protocol][4])
2. User runs “generate release notes.” Tool scans `workspace://repoA` only.
3. Tool hits ambiguity → **Elicitation**: “Which tag range? v1.9…v2.0 or since last commit?” ([Model Context Protocol][6])
4. To draft prose → **Sampling** request: “Client, ask the LLM to summarize these commits into release notes.” (Client may show a confirmation UI.) ([Model Context Protocol][1])

[1]: https://modelcontextprotocol.io/docs/concepts/sampling?utm_source=chatgpt.com "Sampling"
[2]: https://modelcontextprotocol.info/docs/concepts/sampling/?utm_source=chatgpt.com "Sampling - Model Context Protocol （MCP）"
[3]: https://www.dailydoseofds.com/model-context-protocol-crash-course-part-5/?utm_source=chatgpt.com "Integrating Sampling into MCP Workflows"
[4]: https://modelcontextprotocol.io/docs/concepts/roots?utm_source=chatgpt.com "Roots"
[5]: https://modelcontextprotocol.info/docs/concepts/roots/?utm_source=chatgpt.com "Roots - Model Context Protocol （MCP）"
[6]: https://modelcontextprotocol.io/docs/concepts/elicitation?utm_source=chatgpt.com "Elicitation"
[7]: https://www.dailydoseofds.com/model-context-protocol-crash-course-part-8/?utm_source=chatgpt.com "Practical MCP Integration with 4 Popular Agentic Frameworks"
[8]: https://gofastmcp.com/clients/elicitation?ref=dailydoseofds.com&utm_source=chatgpt.com "User Elicitation"
