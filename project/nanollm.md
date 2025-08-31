## Deep Project Understanding: nano-vllm Investigation Roadmap

### Step-by-Step Investigation Roadmap

1. **Entry Point Discovery**: Traced startup (LLMEngine init, process spawning, model loading) → core runtime loop (generate() while loop, step() orchestration) → shutdown (atexit exit, process joins, resource cleanup). Key phases: Init (config, TP setup), Running (scheduling, inference), Queue Management (batching, preemption), Hangup (graceful deallocation).
2. **State Machine Analysis**: Mapped sequence lifecycle (WAITING → RUNNING → FINISHED) with transitions via Scheduler.schedule() and postprocess(). Persistent state: KV cache blocks; ephemeral: sequence metadata. Recovery via preemption; minimal error handling.
3. **Feature Starpoints**: Identified 4 critical modules (LLMEngine, Scheduler, ModelRunner, BlockManager) defining identity through efficient batching, prefix caching, TP, CUDA graphs.
4. **Async Flow Mapping**: Tracked multi-process spawning, shared memory/events for coordination, KV cache contention, implicit error propagation. Bottlenecks: shared memory size, event blocking.
5. **Actionable Experiments**: Proposed 3 safe interventions (logging, param tweaks, graph toggling) for validation.

### Architecture Map (Text Diagram)

```
[User Request] → [LLMEngine]
    ↓              ↓
[Config Load]   [Scheduler] ←→ [BlockManager] (KV Cache Reuse)
    ↓              ↓              ↓
[Tokenizer]   [ModelRunner] ←→ [Sequence] (State Mgmt)
    ↓              ↓              ↓
[Disk I/O]   [CUDA Graphs]   [Shared Memory] (TP Sync)
    ↓              ↓              ↓
[Shutdown]   [Process Join]   [Resource Dealloc]
```

### Core Module Profiles

#### 1. LLMEngine

**Role**: Orchestrates inference pipeline, manages TP processes, ensures scalable GPU utilization.  
**State Dependencies**: Sequence statuses, global engine state.  
**Call Graph**: Upstream: User generate/add_request; Downstream: Scheduler, ModelRunner.  
**Async Behavior**: Spawns processes, uses barriers for sync.  
**I/O Footprint**: Disk (model/tokenizer), shared memory.  

```
function inference_orchestrator(input: request_list):
    validate(input) -> config_check
    business_rule_application -> multi_process_spawn
    side_effects -> distributed_barrier_sync
    return generated_outputs
```

#### 2. Scheduler

**Role**: Optimizes batching for prefill/decode, handles preemption for fairness.  
**State Dependencies**: Sequence queues, block allocations.  
**Call Graph**: Upstream: LLMEngine step; Downstream: BlockManager, Sequence.  
**Async Behavior**: Synchronous batching, enables concurrency via grouping.  
**I/O Footprint**: In-memory queues.  

```
function batch_scheduler(input: sequence_queue):
    validate(input) -> resource_limits_check
    business_rule_application -> prefill_decode_batching
    side_effects -> kv_cache_allocation
    return scheduled_sequences
```

#### 3. ModelRunner

**Role**: Executes optimized inference with CUDA graphs, manages TP sharding.  
**State Dependencies**: Token IDs, KV tensors.  
**Call Graph**: Upstream: LLMEngine call; Downstream: Model, Sampler.  
**Async Behavior**: Process parallelism, graph replay.  
**I/O Footprint**: Disk (weights), CUDA memory.  

```
function model_executor(input: batched_sequences):
    validate(input) -> cuda_device_check
    business_rule_application -> graph_capture_inference
    side_effects -> distributed_sync
    return sampled_tokens
```

#### 4. BlockManager

**Role**: Enables prefix caching via hash-based reuse, manages memory blocks.  
**State Dependencies**: Block tables, ref counts.  
**Call Graph**: Upstream: Scheduler; Downstream: Sequence.  
**Async Behavior**: Synchronous allocation, supports concurrent reuse.  
**I/O Footprint**: GPU memory pools.  

```
function cache_manager(input: sequence_blocks):
    validate(input) -> hash_collision_check
    business_rule_application -> prefix_reuse_allocation
    side_effects -> ref_count_update
    return allocated_blocks
```

### Experimental Validation

1. **Logging for Transitions**: Add logs in Scheduler for WAITING→RUNNING→FINISHED; observe batch sizes and bottlenecks.
2. **Parameter Tweak**: Reduce max_num_batched_tokens; measure throughput impact to validate batching logic.
3. **Async Modification**: Toggle CUDA graphs; compare latency to confirm decode efficiency.

Differentiating features: Lightweight TP, prefix caching, CUDA graphs for high-throughput inference. Actionable: Optimize shared memory, add error handling, monitor KV thrashing.
