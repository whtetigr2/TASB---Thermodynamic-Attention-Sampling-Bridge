# Thermodynamic Attention Sampling Bridge (TASB)

**A validated zero-retraining bridge that preserves frozen-transformer behavior on a thermodynamic sampling substrate.**

TASB encodes transformer attention as an Ising energy-based model, samples it with Extropic's `thrml` stack, and reconstructs the attention pathway inside a live model forward pass without modifying model weights.

This is already producing measurable value, not just conceptual alignment: in the current GPT-2 validation sweep, TASB preserved the vanilla next token on **7/7 tested sequence lengths**, passed the causal mask on **84/84 head checks**, reached aggregate output cosine as high as **0.916**, and beat naive **uniform** and **shuffle** baselines on the strongest tested in-envelope and mid-SNR prompts.

## Why This Matters

- No retraining
- No weight modification
- Live transformer integration
- Structured advantage over naive public baselines
- Regime-aware behavior with a measured operating envelope

## What TASB Is

TASB is a compatibility layer for frozen transformers. It does not retrain the model, replace the language head, or redefine the network. It intercepts attention during inference, converts the attention distribution into an Ising-style energy-based model, samples that model with `thrml`, and reconstructs the original attention pathway back into the live forward pass.

At a high level:

1. Read a frozen attention head from a live transformer forward pass.
2. Convert the softmax attention row into an Ising EBM.
3. Sample that EBM with Gibbs chains through `thrml`.
4. Reconstruct the context vector with the transformer's own value and output projection matrices.
5. Return the modified attention output to the model.

The result is not a new model family. It is a bridge between existing transformer inference and a thermodynamic sampling substrate.

## What Has Been Demonstrated

Current evidence supports a clear, narrower claim:

- TASB can preserve next-token behavior in meaningful tested regimes.
- TASB can outperform naive `uniform` and `shuffle` baselines where structured encoding matters.
- TASB has a measurable operating envelope rather than a vague "works everywhere" story.
- TASB benefits from gating and fallback logic in fragile decision-boundary regimes.

That is already a real result. It is stronger and more credible than claiming universal exactness before hardware validation is complete.

## Key Validation Points

From the current GPT-2 validation and sweep runs:

- `7/7` token matches across the tested final validation sequence lengths
- `84/84` causal-mask passes in the per-head validation sweep
- aggregate cosine up to `0.916`
- bridge KL below both `uniform` and `shuffle` on the strongest tested in-envelope and mid-SNR prompts
- clear regime split between stable prompts and uncertainty-sensitive prompts

Examples from the recent runs:

- In-envelope prompt (`seq_len=18`): bridge KL `0.021751` vs uniform `0.022329` vs shuffle `0.064136`
- Mid-SNR prompt (`seq_len=8`): bridge KL `0.067253` vs uniform `0.069793` vs shuffle `0.181185`

Those are the kinds of comparisons that matter publicly, because they show value against known baselines rather than only against theory.

## Theory, Carefully Stated

Kim (2026) provides the motivating correspondence: scaled dot-product softmax attention has a formal relationship to canonical-ensemble statistics. TASB starts from that correspondence.

But TASB is not just a direct transcription of Kim's result. The current bridge adds practical engineering heuristics to make the sampler behave well under real finite-budget conditions, including:

- sparse attention-weighted couplings
- entropy-calibrated beta selection
- adaptive thermal-noise control
- sequence-length field scaling
- probability-biased initialization
- contrast amplification
- confidence-aware gating

So the accurate description is:

**TASB is an approximation scheme, grounded in the softmax/Boltzmann correspondence, tuned to preserve attention-like behavior on a thermodynamic sampling substrate.**

That framing is both stronger and safer than overstating exact equivalence where the current implementation clearly includes additional control logic.

## Why `thrml` Is Central

TASB intentionally keeps Extropic's `thrml` sampler as the substrate of record.

That matters because the project is not only about expressing a probabilistic model elegantly. It is about preserving the sampling semantics most closely aligned with the target thermodynamic hardware direction. For that reason:

- the `thrml` path is treated as hardware-faithful core infrastructure
- most optimization work is pushed into the shell around that core
- diagnostics, gating, testing, and sweep logic are designed to be simplified without rewriting the sampler substrate

This is a deliberate design choice, not an implementation accident.

## Current Scope

What the current code and experiments support:

- GPT-2 bridge implementation with live hook-based replacement
- Ising encoding and sampling via Extropic's public `thrml` library
- empirical validation across multiple prompt regimes and sequence lengths
- modular split between sampler core, diagnostics, and gating logic
- confidence-aware gating for fragile regimes

What is not claimed as complete:

- physical TSU hardware validation
- a formal Shannon channel-capacity proof
- an exact end-to-end implementation of Kim's softmax/Boltzmann correspondence
- proof that every current heuristic transfers unchanged to silicon

## Why Gating Exists

The newer gating work was added for a simple reason: attention fidelity metrics alone were not enough.

In ambiguous prompts, small attention perturbations can flip the final token even when KL or cosine still look reasonable. That means a bridge that performs well in stable regimes still needs protection when the decision boundary is flat.

The gating layer therefore tries to:

- bypass the bridge when the token decision is too fragile
- blend bridge and vanilla behavior in medium-risk cases
- reject harmful outputs before committing them

This is control logic around the sampler, not a claim that the sampler is universally safe in every regime.

## Where TASB Looks Strongest

TASB currently looks strongest when:

- the prompt sits inside the measured operating envelope
- the model decision is not razor-thin
- the bridge is compared against public naive baselines
- output preservation matters more than exact full-distribution identity

That is a good place to be. It means the project already has a concrete "works here, degrades here, needs gating here" story instead of a hand-wavy demo-only story.

## What This Repo Is For

This repo is most relevant if you care about:

- thermodynamic or probabilistic hardware for inference
- zero-retraining compatibility layers for frozen transformers
- attention-as-energy-model experiments
- software bridges to future sampling hardware

It is less relevant if you are looking for:

- a polished production inference runtime
- proof that transformer inference has already been deployed on physical TSU hardware
- a finished public benchmark suite with broad model coverage

## Roadmap

Near-term:

- finish wiring the modular shell cleanly around the fixed `thrml` core
- speed up sweeps and gating tests by using true lean-path execution
- improve observability for long `thrml` runs

Next:

- broader model validation
- cleaner benchmark scripts and result summaries
- sharper documentation of the operating envelope and failure modes

Long-term:

- physical validation against real thermodynamic hardware

## References

- Kim (2026), softmax/Boltzmann correspondence
- Extropic `thrml` repository: [github.com/extropic-ai/thrml](https://github.com/extropic-ai/thrml)
- Extropic hardware paper: [arXiv:2510.23972](https://arxiv.org/abs/2510.23972)

## Status

Research project. Active development. Hardware validation pending.

If you are reading this as a technical reviewer, the safest one-line summary is:

**TASB is a working experimental bridge that already preserves transformer behavior in meaningful regimes, uses a thermodynamic sampling substrate directly, and has empirical evidence strong enough to justify serious follow-on validation.**
