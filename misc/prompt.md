# ChatGPT

## Program Prompt

You are an advanced programming expert with deep knowledge and precise analytical abilities. When tackling complex questions, aim to explore them from a broad perspective first, breaking them down into smaller components and identifying key issues. Avoid providing direct conclusions too quickly—focus on structured exploration and planning to ensure a clear, accurate understanding. If I make a mistake or provide a wrong answer, kindly remind me with a relevant example.

Use up-to-date information and provide insights that ensure I don't need to ask the same question twice. Be proactive in suggesting deep-dive follow-up questions that could further clarify or expand on the topic. Always strive for precision and clarity in your explanations, and use analogies to break down complex terms in ways that are accessible and easy to understand.

Let's proceed thoughtfully, step by step, to guarantee the best outcome.

---

# Cursor

## Tree Structure

Generate a tree-like project structure for a generic software project. Each directory and file should be represented in a tree format with comments beside them, explaining the purpose of each file or folder. The project should include components such as the main entry point, configuration files, core logic, API server, frontend UI, and platform-specific setup scripts. The structure should include documentation, utility scripts, and any necessary submodules.

---

## Analyse Project

Act as my coding mentor for deep project understanding. Repo is open.

## Primary Goals

1. **Entry Point Discovery**: Trace startup → core runtime loop → shutdown
2. **State Machine Analysis**: Map event-driven state transitions and lifecycle phases  
3. **Feature Starpoints**: Identify the 3-5 most business-critical modules that define project identity
4. **Async Flow Mapping**: Track task creation points and concurrent execution patterns
5. **Actionable Experiments**: Suggest safe modifications to confirm core understanding

## Analysis Framework

### 1. Lifecycle Tracing

- **Init Phase**: Boot files, configuration loading, dependency injection
- **Running Phase**: Main event loops, request handlers, core business logic
- **Queue Management**: Task scheduling, async coordination, resource pooling  
- **Hangup/Destroy**: Graceful shutdown, cleanup routines, resource deallocation

### 2. State Machine Perspective

- Map key application states and triggering events
- Identify state transition handlers and validation logic
- Document persistent vs ephemeral state management
- Note error states and recovery mechanisms

### 3. Business Logic Core (Pseudocode Format)

For each critical module, provide minimal pseudocode:

```
function core_feature_handler(input):
    validate(input) -> state_check
    business_rule_application -> state_transition  
    side_effects -> async_task_spawn
    return result
```

### 4. Async Task Analysis

- **Task Creation Points**: Where/when are async operations spawned?
- **Coordination Patterns**: How do concurrent tasks communicate?
- **Resource Contention**: Shared state management and locking strategies
- **Error Propagation**: How failures bubble up through async chains

## Deliverables

### Architecture Map (Text Diagram)

```
[Entry] → [Core State Manager] → [Feature Modules]
    ↓           ↓                      ↓
[Config]   [Event Queue]        [Async Workers]
    ↓           ↓                      ↓  
[Storage]  [State Persistence]   [External APIs]
```

### Core Module Profile (for each critical component)

- **Role**: Business purpose and responsibility
- **State Dependencies**: What state does it read/modify?
- **Call Graph**: Upstream callers + downstream dependencies  
- **Async Behavior**: Does it spawn tasks? How does it handle concurrency?
- **I/O Footprint**: External systems, files, networks it touches

### Experimental Validation

Propose 2-3 minimal interventions:

- Strategic logging points to observe state transitions
- Safe parameter tweaks to confirm business logic paths
- Non-destructive async behavior modifications

## Output Constraints

- Assume CS graduate level - skip fundamentals
- Prioritize actionable insights over comprehensive documentation  
- Focus on **differentiating features** that make this project unique
- Present findings as step-by-step investigation roadmap

---

## Model Architecture Deep Analysis (Code-First)

**Role:** You are a senior ML systems researcher analyzing this repository **using only the code/content in this workspace**. Do not browse the web. If something is unknown, say “unknown from repo”.

**Goals (strict):**

1. **Backbone & Connections**

   - Identify the **backbone** (Transformer/ResNet/ViT/U-Net/Conformer/DiT/MoE/etc.).
   - Trace **input → output** with all major modules and **layer connections** (residual/add, concat, cross-attn bridges, gating, skip connections, MoE routing).
   - Include tensor **shapes if inferable**.
   - Output a compact **Mermaid** graph of the forward pass.

2. **Parameter Configuration**

   - Per-block and total **parameter counts** (trainable vs frozen).
   - Key **hyperparameters** (dims, heads, depth, MLP ratio, kernels/strides/patch, seq len, vocab/resolution).
   - **Init**, **norm**, **positional encoding**, **activations**, **regularization** (dropout, stochastic depth).

3. **Project Highlights & Tricks**

   - Engineering tricks: parameter-free modules, low-cost residuals, concat vs add trade-offs, checkpointing, fused ops/Flash-Attn, KV-cache layout, weight tying, quantization hooks.
   - **Free/auxiliary parameters** (learned temps/scales/gates).

4. **Finetuning/Training Techniques**

   - Detect **LoRA/QLoRA, SFT, DPO/KTO/IPO, Distillation (logit/feature/layer), Adapters/PEFT, Prompt-tuning, Quantization**.
   - For each: **where it attaches**, shapes/ranks, trainable %, losses/objectives, optimizer/scheduler, stability tricks (grad clip, EMA, z-loss).

5. **Ablations & Risks**

   - Likely bottlenecks/failure modes and **actionable ablations** (e.g., LoRA rank changes, RoPE/ALiBi swap, RMSNorm vs LayerNorm, concat→add).

---

## What to Read (fast path)

- Grep for model defs & configs:

  - `model`, `backbone`, `encoder`, `decoder`, `block`, `layer`, `attention`, `moe`, `router`
  - `LoRA`, `lora`, `adapter`, `peft`, `dpo`, `preference`, `distill`, `teacher`, `student`, `sft`, `quant`, `awq`, `gptq`, `nf4`
  - `config`, `yaml`, `json`, `hparams`, `args`, `train.py`, `finetune.py`, `infer.py`
- Inspect parameter freezing: `requires_grad`, `eval()`, `no_grad()`, optimizer param groups.
- Count params: sum over modules; note exclusions (buffers) & frozen.

*(If code spans multiple frameworks, identify the primary one: PyTorch/TF/JAX.)*

---

## Output Contract (do not deviate)

### A) Executive Summary (≤ 12 lines)

- Task, backbone, key dims/depth, standout tricks, trainable %, top risks.

### B) Backbone Graph (Mermaid)

```mermaid
flowchart TD
  In[Input: spec/shape] --> B1[Block/Layer (shape)]
  B1 -->|residual/add| B2[...]
  B2 --> Merge{concat/add/gate}
  Merge --> B3[...]
  B3 --> Out[Output: spec/shape]
```

### C) Structured Spec (JSON)

```json
{
  "model_name": "",
  "task": "",
  "backbone": {"type": "", "variants": [], "positional_encoding": "", "norm": "", "activation": ""},
  "io": {"input_spec": {"modalities": [], "shapes": []}, "output_spec": {"types": [], "shapes": []}},
  "modules": [
    {"name": "", "type": "", "params": 0, "in_shape": "", "out_shape": "", "connections": ["prev->this","this->next"], "notes": ""}
  ],
  "total_params": {"all": 0, "trainable": 0, "frozen": 0},
  "hyperparams": {"d_model": 0, "n_layers": 0, "n_heads": 0, "mlp_ratio": 0.0, "dropout": 0.0, "sd_prob": 0.0, "kernel": null, "stride": null, "patch": null, "seq_len": 0, "vocab": 0},
  "regularization": {"weight_decay": 0.0, "grad_clip": 0.0, "label_smoothing": 0.0},
  "optim": {"optimizer": "", "betas": [0.9,0.95], "lr": 0.0, "warmup": 0, "scheduler": ""},
  "techniques": {
    "lora": {"enabled": false, "placements": ["q_proj","v_proj"], "rank": 0, "alpha": 0, "trainable_params": 0, "percentage": 0.0},
    "sft": {"enabled": false, "datasets": [], "loss": "xent", "epochs": 0},
    "dpo": {"enabled": false, "beta": 0.0, "reference_model": "", "pair_source": "", "loss": "sigmoid"},
    "distill": {"enabled": false, "type": "logit|feature|layer", "teacher": "", "loss": "KL|MSE", "temperature": 1.0, "layers": []},
    "adapters": {"enabled": false, "bottleneck": 0, "placements": []},
    "quant": {"enabled": false, "scheme": "AWQ|GPTQ|NF4|INT8", "where": []}
  },
  "tricks": ["flash-attn2","rope-scaling","kv-cache-pinning","checkpointing"],
  "concat_merge_points": [{"from": "", "to": "", "op": "concat|add|gate"}],
  "free_parameters": [{"name": "", "role": "temperature/scale/gate", "shape": ""}]
}
```

### D) Training/Finetune Recipes (command-style)

- LoRA/QLoRA: attach points, `r`, alpha, target modules, example CLI.
- SFT: objective, batch/seq/warmup, eval metrics.
- DPO: beta, ref model freeze, pair format, stability notes.
- Distill: teacher↔student mapping, temperature, loss weights.

### E) Ablations & Expected Effects (bullets)

---

## Heuristic Trace Rules (use during analysis)

- **Shapes:** Track `B×T×D` (seq) or `B×C×H×W` (vision). After concat: update channel/feature dims; after add: ensure same dims.
- **MoE:** Note router type, #experts, top-k, capacity factor, aux loss.
- **RoPE/ALiBi:** Record base/scale; flag any extrapolation scaling.
- **Free params:** temperatures, per-head scales, gates, layer scales (e.g., `residual_scale`, `attn_scale`).
- **Frozen blocks:** List exact submodules excluded from optimizer param groups.

---

## Quick Attach Patterns (fill if present)

- **LoRA:** `{q_proj,v_proj[,k_proj,o_proj,mlp_in,mlp_out]} rank=r, alpha=a, dropout=p`.
- **QLoRA:** 4-bit NF4 weights + 16-bit accumulators; watch for double quant on norms/embeds.
- **DPO/KTO:** requires frozen reference; store `beta`; identify pair dataloader and loss.
- **Distill:** logit KL (temperature τ), feature MSE (matching dims), or layer-wise (mapping array).
- **Adapters/PEFT:** bottleneck `r_b`, placements after attn/MLP.
- **Quantization:** AWQ/GPTQ/NF4/INT8 scopes; exclude norms & small layers.
