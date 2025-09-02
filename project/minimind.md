### A) Executive Summary (≤ 15 lines)

MiniMind is a custom causal language model for efficient text generation with reasoning capabilities. Backbone: Transformer-Decoder-Only with 8 layers, 512 hidden, RMSNorm, RoPE, Flash Attention, and optional MoE. Standout tricks: MoE routing with auxiliary loss, reasoning distillation via special tokens, LoRA for 0.2% trainable params. Total ~14M params, trainable ~28K (LoRA). Key risks: MoE load imbalance, distillation overfitting to reasoning tokens, potential divergence in DPO without reference model tuning.

### B) Backbone Graph (Mermaid)

```mermaid
flowchart TD
  In[Input: input_ids (B, seq_len)] --> Emb[Embedding (vocab=6400, d=512)]
  Emb --> Pos[RoPE Positional Encoding]
  Pos --> L1[Layer 1: Attention + MLP (d=512)]
  L1 -->|residual| L2[Layer 2: Attention + MLP]
  L2 -->|residual| L3[Layer 3: Attention + MLP]
  L3 -->|residual| L4[Layer 4: Attention + MLP]
  L4 -->|residual| L5[Layer 5: Attention + MLP]
  L5 -->|residual| L6[Layer 6: Attention + MLP]
  L6 -->|residual| L7[Layer 7: Attention + MLP]
  L7 -->|residual| L8[Layer 8: Attention + MLP]
  L8 --> Norm[RMSNorm (d=512)]
  Norm --> LM[LM Head (vocab=6400)]
  LM --> Out[Output: logits (B, seq_len, vocab)]
  L1 -->|MoE Gate| Moe[MoE Routing (num_experts=8)]
  Moe --> FF[FeedForward Experts (d=512)]
  FF -->|aux_loss| Loss[Auxiliary Loss]
  L1 -->|LoRA| Lora[LoRA Adapters (r=8)]
  Lora --> Att[Attention Projections]
```

### C) Structured Spec (JSON)

```json
{
  "model_name": "MiniMind",
  "task": "Causal Language Modeling",
  "backbone": {
    "type": "Transformer-Decoder-Only",
    "variants": ["MoE", "Flash Attention"],
    "positional_encoding": "RoPE",
    "norm": "RMSNorm",
    "activation": "SiLU"
  },
  "io": {
    "input_spec": {
      "modalities": ["text"],
      "shapes": ["(B, seq_len)"]
    },
    "output_spec": {
      "types": ["logits"],
      "shapes": ["(B, seq_len, vocab_size)"]
    }
  },
  "modules": [
    {
      "name": "Embedding",
      "type": "Token Embedding",
      "params": 3276800,
      "in_shape": "(B, seq_len)",
      "out_shape": "(B, seq_len, 512)",
      "connections": ["input->this", "this->RoPE"],
      "notes": "Vocab 6400"
    },
    {
      "name": "Attention",
      "type": "Multi-Head Self-Attention",
      "params": 1048576,
      "in_shape": "(B, seq_len, 512)",
      "out_shape": "(B, seq_len, 512)",
      "connections": ["RoPE->this", "this->MLP"],
      "notes": "8 heads, Flash Attention optional"
    },
    {
      "name": "MLP/MoE",
      "type": "FeedForward/Mixture-of-Experts",
      "params": 1572864,
      "in_shape": "(B, seq_len, 512)",
      "out_shape": "(B, seq_len, 512)",
      "connections": ["Attention->this", "this->next_layer"],
      "notes": "8 experts, auxiliary loss for balance"
    },
    {
      "name": "LM Head",
      "type": "Linear Projection",
      "params": 3276800,
      "in_shape": "(B, seq_len, 512)",
      "out_shape": "(B, seq_len, 6400)",
      "connections": ["RMSNorm->this", "this->output"],
      "notes": "Tied weights optional"
    }
  ],
  "total_params": {
    "all": 14000000,
    "trainable": 28000,
    "frozen": 13972000
  },
  "hyperparams": {
    "d_model": 512,
    "n_layers": 8,
    "n_heads": 8,
    "mlp_ratio": 3.0,
    "dropout": 0.0,
    "sd_prob": 0.0,
    "kernel": null,
    "stride": null,
    "patch": null,
    "seq_len": 1024,
    "vocab": 6400
  },
  "regularization": {
    "weight_decay": 0.01,
    "grad_clip": 1.0,
    "label_smoothing": 0.0
  },
  "optim": {
    "optimizer": "AdamW",
    "betas": [0.9, 0.95],
    "lr": 0.0001,
    "warmup": 100,
    "scheduler": "cosine"
  },
  "techniques": {
    "lora": {
      "enabled": true,
      "placements": ["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
      "rank": 8,
      "alpha": 32,
      "trainable_params": 28000,
      "percentage": 0.2
    },
    "sft": {
      "enabled": true,
      "datasets": ["reasoning_data"],
      "loss": "cross_entropy",
      "epochs": 3
    },
    "dpo": {
      "enabled": true,
      "beta": 0.1,
      "reference_model": "base_model",
      "pair_source": "preference_pairs",
      "loss": "sigmoid"
    },
    "distill": {
      "enabled": true,
      "type": "logit",
      "teacher": "larger_model",
      "loss": "KL",
      "temperature": 2.0,
      "layers": []
    },
    "adapters": {
      "enabled": false,
      "bottleneck": 0,
      "placements": []
    },
    "quant": {
      "enabled": false,
      "scheme": "",
      "where": []
    }
  },
  "tricks": ["flash-attn2", "rope-scaling", "moe-aux-loss", "reasoning-token-masking", "kv-cache-pinning"],
  "concat_merge_points": [
    {
      "from": "attention_output",
      "to": "mlp_input",
      "op": "add"
    },
    {
      "from": "moe_experts",
      "to": "mlp_output",
      "op": "gate"
    }
  ],
  "free_parameters": [
    {
      "name": "moe_gate",
      "role": "expert routing",
      "shape": "(num_experts,)"
    },
    {
      "name": "aux_loss_weight",
      "role": "load balancing",
      "shape": "scalar"
    }
  ],
}
```

### D) Training/Finetune Recipes (command-style)

- **LoRA/QLoRA**: Attach to q/k/v/o_proj, gate/up/down_proj; r=8, alpha=32, target_modules=["q_proj","v_proj"]; CLI: `python trainer/train_lora.py --lora_rank 8 --lr 1e-4 --batch_size 8 --epochs 3 --bf16`.
- **SFT**: Objective: cross_entropy on reasoning data; batch=8, seq_len=1024, warmup=100 steps, eval perplexity.
- **DPO**: Beta=0.1, ref_model=base, pair_format={"chosen":"text","rejected":"text"}; stability: grad_clip=1.0, monitor reward gap.
- **Distill**: Teacher=larger_model, student=MiniMind, loss=KL(temperature=2.0), weights=0.5 for reasoning tokens; match logits only.

### E) Ablations & Expected Effects (bullet list)

- **Change LoRA rank to 4**: Reduce trainable params to 0.1%, expect slight drop in adaptation quality but faster training.
- **Replace RMSNorm with LayerNorm**: Potential instability in long sequences, higher memory for norm stats.
- **Swap RoPE with absolute PE**: Worse extrapolation to longer contexts, possible positional collapse.
- **Remove MoE auxiliary loss**: Imbalanced expert usage, degraded performance on diverse inputs.
- **Disable reasoning token masking in distillation**: Overfitting to general text, reduced chain-of-thought coherence.
- **Increase DPO beta to 0.5**: Stronger preference alignment but risk of reward hacking and divergence.
