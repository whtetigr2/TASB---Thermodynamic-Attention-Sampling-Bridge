# Thermodynamic Attention Sampling Bridge (TASB)

Zero-retraining bridge for running frozen transformer attention through a thermodynamic sampler.

TASB takes a transformer attention distribution, encodes it as an Ising-style energy-based model, samples that model with Extropic's `thrml` stack, and reconstructs the attention output back into the transformer forward pass.

The project goal is not "train a new thermodynamic model." The goal is to preserve useful transformer behavior on a thermodynamic sampling substrate without modifying model weights.

## What TASB Does

At a high level:

1. Read a frozen attention head from a live transformer forward pass.
2. Convert that softmax attention row into an Ising EBM.
3. Sample the EBM with Gibbs chains using `thrml`.
4. Reconstruct the context vector with the transformer's own value and output projection matrices.
5. Return the modified attention output to the model.

This makes TASB a compatibility layer around existing transformers, not a retrained architecture.

## Current Position

What is supported by the current code and experiments:

- GPT-2 bridge implementation with live hook-based replacement
- Ising encoding and sampling via Extropic's public `thrml` library
- empirical validation across multiple prompt regimes and sequence lengths
- gating logic for bypassing or blending in fragile regimes
- modular split between hardware-faithful core and shell logic

What is **not** claimed as complete:

- physical TSU hardware validation
- a formal Shannon channel-capacity proof
- an exact implementation of Kim's softmax/Boltzmann correspondence
- proof that all heuristics transfer unchanged to silicon

## Theory, Carefully Stated

Kim (2026) gives the key motivation: scaled dot-product softmax attention has a formal correspondence to canonical-ensemble statistics. TASB starts from that correspondence.

But TASB is not just a direct transcription of that result. The current bridge adds empirical engineering heuristics to make the sampler behave well in practice, including:

- sparse attention-weighted couplings
- entropy-calibrated beta selection
- adaptive thermal-noise control
- sequence-length field scaling
- probability-biased initialization
- contrast amplification
- confidence-aware gating

So the accurate description is:

TASB is an approximation scheme, grounded in the softmax/Boltzmann correspondence, tuned to preserve attention-like behavior on a thermodynamic sampling substrate.

## Why `thrml` Matters Here

TASB intentionally keeps Extropic's `thrml` sampler as the substrate of record.

That matters because the project is not only about probabilistic modeling elegance. It is about preserving the sampling semantics most closely aligned with the target thermodynamic hardware direction. For that reason, the code treats the `thrml` call path as hardware-faithful core infrastructure and pushes optimization work into the surrounding shell.

## Repository Structure

Core files:

- `bridge_core.py`: hardware-faithful Ising encoding and `thrml` sampler path
- `bridge_diagnostics.py`: fidelity and analysis metrics
- `bridge_gating.py`: confidence-aware gating and acceptance logic
- `tsu_attention_bridge_v5_12.py`: integrated bridge implementation
- `tsu_v512_gating_test.py`: comparison harness for v5.11 vs v5.12 gating behavior

Design boundary:

- `bridge_core.py` is the place to preserve sampler fidelity.
- diagnostics, gating, testing, sweep logic, and performance work belong outside the core whenever possible.

## Empirical Results

So far, the project supports a narrower but still meaningful claim:

- the bridge can preserve next-token behavior well in some regimes
- it performs best in an intermediate operating envelope
- uncertain prompts are much more fragile and need gating or fallback
- the bridge is better described as regime-dependent than universally exact

That is a stronger and more defensible story than claiming blanket equivalence across all regimes.

## What the Gating Layer Is For

The v5.12 gating work exists because attention fidelity metrics alone were not enough. In ambiguous prompts, small attention perturbations can flip the final token even when KL or cosine still look reasonable.

The gating layer therefore tries to:

- bypass the bridge when the decision boundary is too fragile
- blend bridge and vanilla behavior in medium-risk cases
- reject harmful outputs before committing them

This is a practical control layer around the sampler, not part of the `thrml` core itself.

## What This Repo Is Best For

This repo is useful if you are interested in:

- thermodynamic or probabilistic hardware for inference
- zero-retraining compatibility layers for frozen transformers
- attention-as-energy-model experiments
- bridging software experiments to a future sampling hardware substrate

It is less useful if you are looking for:

- a polished production inference runtime
- a proof that transformer inference has already been ported to physical TSU hardware
- a finished public benchmark suite

## Roadmap

Near-term:

- finish wiring the modular shell around the fixed `thrml` core
- speed up sweeps and gating tests by using true lean-path execution
- add better run-time observability for long `thrml` jobs

Next:

- broader model validation
- cleaner benchmark scripts and result summaries
- stronger public documentation of operating envelope and failure modes

Long-term:

- physical validation against real thermodynamic hardware

## References

- Kim (2026), softmax/Boltzmann correspondence
- Extropic `thrml` repository: [github.com/extropic-ai/thrml](https://github.com/extropic-ai/thrml)
- Extropic hardware paper: [arXiv:2510.23972](https://arxiv.org/abs/2510.23972)

## Status

Research project. Active development. Hardware validation pending.

If you are reading this as a technical reviewer, the safest summary is:

TASB is a promising experimental bridge that preserves transformer behavior in meaningful regimes, using a thermodynamic sampling substrate, with empirical evidence in software and careful caveats around what has and has not yet been proven.
