## Open-LLM-VTuber Deep Project Understanding: Investigation Roadmap

### Architecture Map (Text Diagram)

```
[Entry Points]
├── run_server.py (Main Server)
├── scripts/run_bilibili_live.py (Live Streaming Client)
└── server.py (WebSocketServer)

[Core Runtime Loop]
├── WebSocketServer (FastAPI + Uvicorn)
│   ├── /client-ws (Client Connections)
│   ├── /proxy-ws (Proxy Mode)
│   └── Static Routes (/live2d-models, /avatars, etc.)
├── WebSocketHandler (Message Routing)
│   ├── Message Types: text-input, mic-audio-end, interrupt-signal, etc.
│   └── Route to Handlers: conversation_trigger, audio_data, config, etc.
└── ServiceContext (Per-Client Initialization)
    ├── ASR Engine (Multiple Providers)
    ├── TTS Engine (Multiple Providers)
    ├── Agent Engine (LLM + MCP Tools)
    ├── Live2D Model (Avatar Animation)
    └── MCP Components (Tool Management)

[State Management]
├── ConversationHandler (Multi-Modal Processing)
│   ├── Single/Group Conversations
│   ├── Interrupt Handling
│   └── Proactive Speak Signals
├── ProxyMessageQueue (Producer-Consumer Pattern)
│   ├── Queue text-input when active
│   └── Process sequentially
└── ChatGroupManager (Group Membership)

[Async Coordination]
├── Task Creation: conversation triggers, danmaku processing
├── Communication: WebSocket broadcasts, message queues
├── Contention: Shared dicts (client_contexts, audio_buffers)
└── Error Propagation: Cancellation, reconnection, cleanup

[External Integrations]
├── BiliBili Live (Danmaku → VTuber)
├── MCP Servers (Tool-Augmented Responses)
├── LLM Providers (OpenAI, Claude, Ollama, etc.)
└── ASR/TTS APIs (Azure, Groq, Local Models)
```

### Core Module Profiles

#### 1. conversation_handler

- **Role**: Orchestrates multi-modal conversation processing, handling text/audio inputs, proactive speak signals, and managing both individual and group conversations with interrupt capabilities.
- **State Dependencies**: Reads/modifies `current_conversation_tasks` (dict of asyncio tasks), `client_contexts` (service contexts per client), `chat_group_manager` (group membership state), `received_data_buffers` (audio data per client).
- **Call Graph**: Upstream callers: `websocket_handler._handle_conversation_trigger` (routes WebSocket messages); Downstream dependencies: `process_single_conversation`, `process_group_conversation`, `handle_individual_interrupt`, `handle_group_interrupt`, `store_message` (chat history).
- **Async Behavior**: Spawns `asyncio.create_task` for conversation processing, handles concurrency through task cancellation and group task management, uses locks implicitly through task state checks.
- **I/O Footprint**: WebSocket (sends JSON messages via `websocket.send_text`), files (chat history via `store_message`), external prompts (loads from `prompts/` directory via `prompt_loader.load_util`).

#### 2. live2d_model

- **Role**: Prepares Live2D avatar animation payloads by managing model information, extracting emotions from text, and providing emotion mapping for real-time facial expressions.
- **State Dependencies**: Reads `model_dict.json` (model configuration), modifies internal `model_info`, `emo_map`, `emo_str` (emotion mappings).
- **Call Graph**: Upstream callers: `service_context.init_live2d` (initializes model), `transformers.actions_extractor` (extracts actions); Downstream dependencies: None direct (only prepares payloads, doesn't send to frontend).
- **Async Behavior**: Synchronous operations, no task spawning or concurrency handling.
- **I/O Footprint**: Files (reads `model_dict.json` and Live2D model files from `live2d-models/` directory with encoding detection).

#### 3. agent_factory

- **Role**: Creates and configures conversational AI agents supporting multiple LLM providers and agent types, enabling tool-augmented responses and personality-driven interactions.
- **State Dependencies**: Reads `agent_settings`, `llm_configs`, `system_prompt`, modifies agent instances with MCP components (`tool_manager`, `tool_executor`).
- **Call Graph**: Upstream callers: `service_context.init_agent` (creates agents from config); Downstream dependencies: Creates `BasicMemoryAgent`, `Mem0LLM`, `HumeAIAgent`, `LettaAgent` instances that use `stateless_llm_factory`, `live2d_model`, MCP components.
- **Async Behavior**: Synchronous factory creation, but created agents handle async LLM calls and tool execution.
- **I/O Footprint**: Networks (indirect via LLM APIs in created agents), files (loads tool prompts from config).

#### 4. asr_factory

- **Role**: Instantiates automatic speech recognition engines for real-time audio-to-text conversion, supporting multiple ASR providers for voice input processing.
- **State Dependencies**: Reads `asr_config` (model and provider settings), creates ASR engine instances.
- **Call Graph**: Upstream callers: `service_context.init_asr` (initializes ASR from config); Downstream dependencies: Creates `FasterWhisperASR`, `WhisperCPPASR`, `WhisperASR`, `FunASR`, `AzureASR`, `GroqWhisperASR`, `SherpaOnnxASR` instances.
- **Async Behavior**: Synchronous factory creation, but ASR engines handle async audio processing.
- **I/O Footprint**: Networks (Azure ASR API, Groq API), files (local model files for `faster_whisper`, `whisper_cpp`, `fun_asr`, `sherpa_onnx`).

#### 5. bilibili_live

- **Role**: Implements live streaming integration with BiliBili platform, receiving danmaku messages from live rooms and forwarding them to the VTuber for real-time interaction.
- **State Dependencies**: Reads `room_ids`, `sessdata` (authentication), modifies connection state (`_connected`, `_running`), message handlers list.
- **Call Graph**: Upstream callers: `scripts/run_bilibili_live.py` (main entry point); Downstream dependencies: `blivedm.BLiveClient` (external library), proxy WebSocket connection, `VtuberHandler` (danmaku processing).
- **Async Behavior**: Spawns `asyncio.create_task` for message receiving loop, handles WebSocket connections and heartbeat monitoring concurrently.
- **I/O Footprint**: Networks (BiliBili Live API via `blivedm`, WebSocket to proxy server), external systems (BiliBili live rooms, proxy WebSocket at `ws://localhost:12393/proxy-ws`).

### Experimental Validation

#### Experiment 1: Strategic Logging for State Transitions in Conversation Handler

**Objective:** Observe state transitions in conversation initiation and interruption to validate lifecycle tracing and state machine behavior.  
**File:** `src/open_llm_vtuber/conversations/conversation_handler.py`  
**Line Range:** 74-109 (group/single conversation start logic) and 146-181 (group interrupt handling).  
**Suggested Modification:** Add temporary logging statements to track state changes without altering logic.  
**Code Snippet:**

```python
# In handle_conversation_trigger, around line 74-82:
logger.info(f"[EXPERIMENT] State transition: Checking group for client {client_uid}, group exists: {group is not None}, members: {len(group.members) if group else 0}")
if group and len(group.members) > 1:
    logger.info(f"[EXPERIMENT] Transitioning to group conversation state for {task_key}")
    # ... existing code ...

# In handle_group_interrupt, around line 159-181:
logger.info(f"[EXPERIMENT] State transition: Interrupting group {group_id}, current speaker: {current_speaker_uid}, task status: {task.done() if task else 'None'}")
# ... existing code ...
GroupConversationState.remove_state(group_id)
logger.info(f"[EXPERIMENT] State cleaned up for group {group_id}")
```

#### Experiment 2: Safe Parameter Tweak for Business Logic Validation in MCP Client

**Objective:** Confirm tool call timeout handling and error paths by adjusting the default timeout to observe impact on business logic without breaking functionality.  
**File:** `src/open_llm_vtuber/mcpp/mcp_client.py`  
**Line Range:** 14 (DEFAULT_TIMEOUT definition) and 57-71 (timeout usage in session creation).  
**Suggested Modification:** Temporarily reduce DEFAULT_TIMEOUT to 15 seconds to test timeout logic.  
**Code Snippet:**

```python
# Around line 14:
DEFAULT_TIMEOUT = timedelta(seconds=15)  # Temporarily reduced from 30 for experiment

# Around line 57-71:
timeout = server.timeout if server.timeout else DEFAULT_TIMEOUT
logger.info(f"[EXPERIMENT] Using timeout: {timeout.total_seconds()}s for server {server_name}")
# ... existing code ...
```

#### Experiment 3: Non-Destructive Async Behavior Modification in Single Conversation Processing

**Objective:** Observe async flow impacts by introducing a minimal delay in the agent output processing loop to validate concurrency and task management.  
**File:** `src/open_llm_vtuber/conversations/single_conversation.py`  
**Line Range:** 92-124 (agent output stream processing loop).  
**Suggested Modification:** Add a small async delay after processing each output item to monitor flow without blocking.  
**Code Snippet:**

```python
# Around line 92-124:
async for output_item in agent_output_stream:
    # ... existing processing ...
    if isinstance(output_item, (SentenceOutput, AudioOutput)):
        response_part = await process_agent_output(...)
        full_response += str(response_part) if response_part else ""
        # Add minimal delay for experiment
        await asyncio.sleep(0.01)  # 10ms delay to observe async flow
        logger.debug(f"[EXPERIMENT] Processed output item, current response length: {len(full_response)}")
    # ... existing code ...
```

This roadmap provides a comprehensive understanding of the Open-LLM-VTuber as a sophisticated VTuber platform integrating real-time audio processing, avatar animation, live streaming, and tool-augmented AI conversations. The differentiating features include multi-modal input handling, concurrent conversation management, and seamless integration with external platforms like BiliBili.
