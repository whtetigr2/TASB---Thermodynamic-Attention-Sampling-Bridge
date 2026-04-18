# Llama Bridge Review Findings — 2026-04-18

This note captures review findings from the recent Llama-based TASB runs and the associated artifacts:

- `tasb_freetext_dashboard.html`
- `tasb_freetext_20260418_030057.csv`
- `sweep_gated_combined_20260418_011721.csv`
- the current Llama full-stack path in `tasb_fullstack.py`
- the current optimized bridge core in `bridge_core_v4.py`

These findings matter because they help separate real progress from claims that still need tighter validation language.

## What Looks Real

The current Llama bridge work does appear to show real architectural and runtime progress:

- end-to-end timings are significantly better than the earlier "hours on one case" behavior
- the bridge path is cleaner and more modular than before
- the module-level runner cache in `bridge_core_v4.py` is a meaningful improvement over the earlier compile pattern
- the current stress runs support a credible "faster and more stable" claim for the live bridge path

Those are real improvements and should be preserved in future writeups.

## Findings

### 1. Adaptive mode does not actually skip low-cos layers yet

**Priority:** P1

`tasb_fullstack.py` describes adaptive mode as:

> Hook all layers, skip any where output_cos < threshold

But the implementation currently does this:

- `target_layers = list(range(NUM_LAYERS))` for adaptive mode
- hooks all target layers each step
- uses the cosine threshold only in reporting, after the run

That means the current outputs do **not** validate the advertised adaptive co-processor behavior or real selective layer bypass.

**Conclusion:**
Current Llama full-stack results can support a claim like:

- "multi-layer bridge path tested"
- "layer cosine measured across the stack"

They should **not** yet be described as proof of active adaptive layer skipping.

### 2. Stage 3 acceptance is stubbed to unconditional success

**Priority:** P1

Both the current full-stack runner and the standalone v4 test patch `run_acceptance_gate` to an unconditional lambda returning `accepted=True`.

That means the current outputs are useful as:

- throughput evidence
- sampler stability evidence
- bridge token-match evidence

But they are **not** yet evidence that the real downstream acceptance gate safely rejects harmful outputs in deployment.

**Conclusion:**
The current Llama results should distinguish between:

- "bridge speedup / bridge stability"
- and
- "validated acceptance-gated deployment"

Only the first is supported by these runs.

### 3. The "one compile per config" wording is still too strong

**Priority:** P2

The new module-level runner cache is a real improvement. However, `_get_runner()` keys the cache on:

- `seq_len`
- `n_burnin`
- `ann_steps`
- `init_ns`

And `thermo_attn_v4()` updates `init_ns` from `thermal_state.get_initial_noise(...)` before fetching the runner.

Since that per-head value can drift over time, the cache can fragment across multiple compiled runners for what is otherwise the same sequence and annealing configuration.

**Conclusion:**
The architecture clearly reduces recompilation overhead. But the safest wording right now is:

- "significantly reduced recompilation overhead"

not:

- "one compile total"
- or
- "one compile per config"

unless `init_ns` is fixed or removed from the runner cache key.

## Claim-Safe Summary

The strongest defensible summary from the current Llama bridge work is:

- TASB has made real progress in runtime and architecture
- the Llama bridge path is materially faster and cleaner than earlier versions
- the current full-stack path supports meaningful multi-layer bridge testing
- the current runs do **not** yet validate true adaptive layer skipping
- the current runs do **not** yet validate a live downstream acceptance-gated deployment path

## Recommended Public Wording

Use language closer to:

> The current Llama bridge stack shows significant runtime and architecture improvement, with a cleaner multi-layer path and materially reduced end-to-end latency. Current results support bridge throughput and behavior-preservation claims, while adaptive skipping and full downstream acceptance gating remain in-progress.

Avoid wording closer to:

- "adaptive co-processor path validated"
- "acceptance-gated deployment validated"
- "one compile total"

## Status

These findings do **not** negate the progress. They narrow the strongest defensible claim so the project stays credible while the Llama path matures.
