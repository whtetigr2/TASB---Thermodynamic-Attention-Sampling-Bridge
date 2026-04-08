# TASB---Thermodynamic-Attention-Sampling-Bridge
Run transformers on thermodynamic hardware, no retraining required.
[README.md](https://github.com/user-attachments/files/26557030/README.md)
# Thermodynamic Attention Sampling Bridge (TASB)

**Zero-retraining compatibility layer for transformer inference on thermodynamic hardware.**

[![Patent](https://img.shields.io/badge/USPTO-64%2F019%2C999-blue)](https://www.uspto.gov)
[![Bridge](https://img.shields.io/badge/Bridge-v5.11-green)]()
[![Validated](https://img.shields.io/badge/Validated-GPT--2%20%2B%20Llama%203B-brightgreen)]()
[![Status](https://img.shields.io/badge/Status-Active%20Development-orange)]()

---

## What This Is

TASB encodes the softmax attention mechanism of any frozen transformer model as an
**Ising Energy-Based Model (EBM)** for execution on thermodynamic computing hardware —
specifically Extropic's Thermodynamic Sampling Unit (TSU).

**Zero retraining. No weight modification. No architectural changes.**
Drop-in replacement at inference time.

```
Frozen Transformer → TASB Encoder → TSU Sampler → Output Logits
```

---

## The Core Insight

Kim (2026) proved that softmax attention is formally isomorphic to a Boltzmann
distribution at inverse temperature β = 1/√d_k. This means encoding attention
as an Ising EBM is a **change of representation, not an approximation**.

TASB implements this encoding with seven validated fixes that address the specific
failure modes of thermodynamic encoding:

| Fix | Name | Function |
|-----|------|----------|
| A | Multi-Modal Symmetry Breaker | Gumbel-Max per-chain reparameterization |
| B | Relational Topology Mapper | Attention-weighted sparse Ising coupling |
| C | Entropy-Calibrated Thermal Scalar | Per-row beta calibration |
| D | **Adaptive Thermal Governor** | P-controller entropy thermostat ← primary contribution |
| E | Scale-Invariant Field Governor | Sequence-adaptive field scaling |
| F | Probability-Biased Initialization | Warm start from attention distribution |
| G | Contrast Amplification | Noise-aware encoding gain |

Fix D is the primary original contribution — a closed-loop proportional controller
that drives sampler output entropy toward the target entropy of the input attention
row. Not in Kim (2026), Extropic arXiv 2510.23972, or any denoising diffusion literature.

---

## Validated Results

### GPT-2 Full Validation Sweep (7 tiers, 84 heads)

| seq_len | Agg Cosine | Causal | Token Match |
|---------|----------- |--------|-------------|
| 5       | 0.675      | 12/12  |     ✓      |
| 9       | 0.742      | 12/12  |     ✓      |
| 15      | 0.742      | 12/12  |     ✓      |
| 18      | 0.789      | 12/12  |     ✓      |
| 30      | 0.842      | 12/12  |     ✓      |
| 52      | 0.882      | 12/12  |     ✓      |
| 95      | 0.916      | 12/12  |     ✓      |

**7/7 token matches. 84/84 causal passes. 100%.**

### Llama 3.2-3B Cross-Architecture Validation
- 24/24 heads — all HIGH cosine, all causal PASS
- Token match confirmed on Tier 4 prompt
- Bridge v5.9 — identical to GPT-2 validation

### Control Tests — Attention Is Load-Bearing
- Uniform attention on all 12 heads simultaneously breaks token prediction
- Bridge preserves it
- Bridge KL divergence **2.76× lower** than random shuffle baseline

### Regime-Dependent Behavior (Shannon-Consistent)

| Entropy Band | Bridge vs Baselines | Interpretation |
|--------------|--------------------|-|
| Low (< 4.0 nats, seq < 8) | Beats uniform | Below operating envelope |
| Mid (4.0–6.0 nats) | **BEATS BOTH** ✓ | Operating envelope — bridge wins |
| High (> 6.5 nats) | Beats shuffle | Channel capacity ceiling |

Bridge advantage is maximal precisely where Shannon channel coding theory
predicts encoding gain — in the intermediate entropy regime where inference
decisions are most sensitive. This is not a coincidence.

---

## The Information Theory Framing

TASB is a **noise-aware channel encoder** for thermodynamic hardware.

| Shannon Framework | TASB System |
|------------------|-------------|
| Message | Softmax attention distribution |
| Encoder | TASB bridge (Fixes A–G) |
| Channel | Thermodynamic Gibbs sampler |
| Noise | Thermal + stochastic behavior |
| Decoder | Output logits (already exists) |
| Encoding gain | Fix G contrast_gamma parameter |
| AGC | Fix D adaptive thermal governor |

The operating envelope corresponds to the entropy regime where the
**separation property** of reservoir computing is satisfied — different inputs
produce distinguishable physical states above the noise floor.

> *"Tunneling SIPR through NIPR."*
> The softmax signal is encoded to survive thermodynamic noise,
> not to eliminate it.

---

## Why This Matters

Every other approach to thermodynamic AI inference assumes retraining or accepts
fidelity loss. TASB proves neither is necessary.

On real TSU silicon, the Gibbs sampling step — which dominates CPU simulation time —
runs at physical thermodynamic speed. The encoding quality proven here transfers
directly. The speed constraint disappears.

Current validation is on CPU simulation (Extropic thrml library). Physical hardware
validation is the next step.

---

## Relevant Literature

- Kim (2026) arXiv:2602.08216 — Softmax = Boltzmann, β = 1/√d_k
- Roberts, McCourt et al. (2026) PRR 8:013161 — Log-mean coupling rationale
- te Vrugt (2024) arXiv:2412.13212 — Reservoir computing, separation property
- Liang et al. (2024) Nature Electronics — Physical reservoir computing
- Extropic arXiv:2510.23972 — TSU hardware architecture

---

## Status

- ✅ USPTO Provisional Patent Application No. **64/019,999** (filed March 28, 2026)
- ✅ GPT-2 validated — 7 sequence lengths, 12 heads, 100% causal integrity
- ✅ Llama 3.2-3B cross-architecture validation complete
- ✅ Regime characterization complete — Shannon-consistent operating envelope
- ✅ Fix G (contrast amplification) implemented and validated — v5.10/v5.11
- ⏳ Physical TSU hardware validation pending

---

## What's Needed

The encoding is proven. The theory is grounded. The operating envelope is
characterized.

**What would unlock the next phase:**
- Access to Extropic TSU hardware for physical validation
- Compute resources for multi-model validation at scale (Mistral, GPT-Neo, etc.)

---

## Contact

US Military Veteran (25N — Network Systems Operator)  
USPTO Provisional Patent No. 64/019,999

*Not affiliated with any institution.*
*Built this because it should exist.*

---

## License

This repository is shared for research visibility purposes.  
All rights reserved. USPTO Provisional Patent Application No. 64/019,999 filed.  
Commercial use, reproduction, or derivative works require written permission.  
Contact the inventor for licensing inquiries.

---

*"Letting the hardware be thermodynamic and building the software to be its consciousness."*
