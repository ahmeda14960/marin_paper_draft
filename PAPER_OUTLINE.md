# PAPER_OUTLINE: Marin (8B + 32B) Fully-Open Training Retrospective

This file is a detailed, paper-shaped plan. Reading it should give a strong overview of (1) what Marin is, (2) what happened in the 8B and 32B runs, (3) which figures/tables support which claims, and (4) where citations belong.

Target style: the “fully open / model flow” genre of OLMo 2 and the OLMo 3 draft in `other_papers/` (process > checkpoint; negative results and operational failures are core scientific outputs).

## 1. One-Sentence Definitions (Use Early, Verbatim)

- **Marin**: a community-driven “fully open” LLM training effort that releases code, data-mixture manifests, intermediate checkpoints, and an issue-driven experiment history in addition to final weights. `\citep{Hall2025marin,marin8bretro,marin32bretro}`
- **8B / 32B**: approximate parameter counts (8B and 32B parameters) of the two main Marin base models, enabling comparisons against contemporary open baselines. `\citep{dubey2024llama,qwen2.5,qwen3,team2025gemma3}`
- **“Issues” (in the retrospectives)**: concrete scale-induced failure modes (loss spikes, attention instability, cached-data contamination, and shuffling pathologies) plus the mitigations that restored reliable training. `\citep{marin8bretro,marin32bretro}`

## 2. Thesis and Contributions (What Readers Should Learn)

Thesis:

- Fully open “model flow” releases enable causal learning from long training runs: you can see what changed, when, and what broke.
- Marin 8B is a case study in long-horizon mid-flight adaptation (cooldowns + reheats + data mix evolution) guided by microannealing probes and stabilized by late-regime tools (EMA monitoring, z-loss). `\citep{marin8bretro}`
- Marin 32B is a case study in scaling a recipe until it breaks: optimizer-side mitigations softened but could not eliminate loss spikes, forcing an architectural fix (QK-norm). After stability, training still failed due to data-system problems (cached contamination and poor shuffling), showing that “systems correctness” remains a first-order science variable. `\citep{marin32bretro}`

Concrete contributions to make explicit in the paper:

- A phase-by-phase, reproducible record for Marin 8B and 32B (phase boundaries, mixes, hyperparameters, evaluation). `\citep{marin8bretro,marin32bretro}`
- Stability lessons:
  - 8B: EMA monitoring (“EMA gap”), deep cooldown behavior, and z-loss stabilizing late decay. `\citep{marin8bretro,palm,chameleon,mitch}`
  - 32B: spike instrumentation + mitigations (grad/update clipping, skip-step), and the decisive mid-run QK-norm switch. `\citep{marin32bretro,Dehghani2023ScalingVT,marinissue1368,marinissue1390}`
- Shuffle correctness case study:
  - why an affine/LCG permutation is a bijection yet a poor shuffle in structured corpora
  - how it produced a cooldown “phase shift” at ~190k steps
  - why a Feistel PRP (statelessly implemented in Levanter) fixed it. `\citep{marin32bretro,levanterprp}`
- Contamination object lesson:
  - cached preprocessing can preserve leakage even after code is “fixed”
  - contamination can harm benchmark scores via prompt-format mismatch (GSM8K in Bison). `\citep{marin32bretro}`

## 3. Paper Skeleton (Maps to LaTeX Files)

- `sections/intro.tex`: motivation, definitions, contributions, paper roadmap.
- `sections/related.tex`: open-weight vs fully-open, midtraining/cooldown, stability tooling, web-scale corpora, evaluation/contamination.
- `sections/methods.tex`: open artifacts, data sources, architectures, optimizer/schedules, infrastructure, shuffling PRPs.
- `sections/experiments.tex`: 8B and 32B retrospectives as empirical case studies; reproduce key plots; reproduce data mix tables.
- `sections/conclusion.tex`: limitations and distilled lessons.

## 4. Quick Reference: Phase Tables

Marin 8B phases (single long run; boundaries assigned retrospectively). `\citep{marin8bretro}`

| Phase | Tokens (approx) | Hardware | Batch (tokens/step) | Why It Matters |
| --- | --- | --- | --- | --- |
| Kestrel | 0 → 2.7T | 2x v5e-256 (multislice) | 4Mi | DCLM mix + WSD-S as a training-time diagnostic schedule |
| Ocelot | ~2.7T → 3.78T | v4-2048 | 12Mi | LR/batch scaling + EMA monitoring; RoPE mismatch fix |
| Jellyfish | 3.78T → 4.78T | v4-2048 | 12Mi | 1T-token cooldown; HQ mix; Paloma formatting sensitivity emerges |
| Phoenix | 4.78T → 11.1T | v4-2048 | 12Mi | Reheat + mixture transition to Nemotron-CC; smooth interpolation |
| Starling | ~11.1T → ~12.75T | v4-2048 | 16Mi | Deep cooldown with z-loss; new markdownified datasets + science QA |

Marin 32B phases (Phase 2 diagnostic restarts excluded from “final artifact”). `\citep{marin32bretro}`

| Phase | Tokens (approx) | Steps | Architecture | Why It Matters |
| --- | --- | --- | --- | --- |
| 1 | 2.679T | 0 → 80k | Llama-3-style attention | Loss spikes; step skipping and clipping not sufficient |
| 2 | (discarded) | (diagnostic) | Llama-3-style attention | Necromancy + Muon attempts; not the final trajectory |
| 3 | 2.684T | 80k → 160k | Qwen3-style attention + QK-norm | Architectural stability fix; spikes disappear after ~10B-token recovery |
| 4 | 1.074T | 160k → 192k | QK-norm | Cooldown failures (contamination + shuffle) and fixes (MegaMath + Feistel PRP) |

## 5. Evidence Map: Figures and Tables

### 5.1 Figures (All from Retros; Stored in `figures/`)

Marin 8B figures:

- `marin-timeline.png`: phase partitioning and token ranges.
- `tootsie-8b-retro-wsd-interval.png`: WSD-S decay-cycle spacing change at ~step 200k.
- `tootsie-8b-wsd-s-loss-transition.png`: eval/training loss drops during decay.
- `tootsie-8b-wsd-s-losses-post-transition.png`: diverse eval-loss trajectories; formatting sensitivity.
- `tootsie-8b-ema-gap.png`: “EMA gap” stability signal.
- `8b-phoenix-transition.png`: smooth Nemotron-CC transition.
- `8b-raccoon-loss-increase.png`: deep cooldown loss creep.
- `marin-8b-spoonbill-loss.png`: Spoonbill loss creep reproduction.
- `8b-spoonbill-norms.png`: lm_head norm explosion evidence.
- `8b-spoonbill-zloss.png`: z-loss fix.
- `marin-8b-starling-c4en.png`: Paloma c4_en perplexity shift.
- `tootsie-8b-starling-loss-slowdown.png`: low-LR slowdown.

Marin 32B figures:

- `32b-spiky-loss.png`: Phase 1 loss spikes.
- `32b-update-spike-precede-loss-spike.png`: update spikes precede loss spikes.
- `32b-loss-comparisons.png`: eval loss comparisons.
- `32b-bad-spike.png`: restart/necromancy spike.
- `32b-muon-vs-adam.png`: Muon vs Adam.
- `32b-muon-can-into-space.png`: Muon divergence.
- `qk-recovery.png`: QK-norm warm-start recovery.
- `32b-shuffle-spike.png`: Bison cooldown phase shift (train loss changes; eval stable).
- `32b-feistel-vs-lcg.png`: Feistel removes phase shift.
- `32b-paloma-c4-en-permutation.png`: Paloma c4_en improves.
- `32b-paloma-average-permutation.png`: Paloma average improves.

### 5.2 Tables (Core Tables to Include)

- 8B: Kestrel mix table; Jellyfish mix table; Starling cooldown mix table (with oversampling); 8B base results; 8B SFT results.
- 32B: pretraining mix table; batch schedule table; Bison cooldown mix table; Mantis cooldown mix table; 32B base results.

## 6. Section-by-Section Outline (Detailed)

## 6.1 Introduction (`sections/intro.tex`)

- Motivation: open weights alone are insufficient for reproducibility and for learning from failure modes.
- Position Marin alongside fully-open releases (OLMo/OLMo 2, Pythia, LLM360, DCLM). `\citep{Groeneveld2024OLMoAT,olmo20242olmo2furious,biderman2023pythia,liu2023llm360,dclm}`
- Define key terms used later:
  - cooldown, reheat, microannealing
  - “phase shift” (train loss shifts while eval stable)
  - PRP shuffle vs “valid permutation”
- Preview the two case studies:
  - 8B: long run with 2 cooldowns + reheat; data mixture evolution; microannealing guidance. `\citep{marin8bretro}`
  - 32B: spikes → QK-norm switch; cooldown failures → contamination + shuffle fix. `\citep{marin32bretro}`
- Contributions list (Section 2).

## 6.2 Related Work (`sections/related.tex`)

- Open-weight baselines (comparators but typically not fully open): Llama, Qwen, Gemma. `\citep{dubey2024llama,qwen2.5,qwen3,team2025gemma3}`
- Fully open training releases: OLMo, OLMo 2, Pythia, LLM360, DCLM. `\citep{Groeneveld2024OLMoAT,olmo20242olmo2furious,biderman2023pythia,liu2023llm360,dclm}`
- Web-scale corpora and curation:
  - Dolma (open multi-trillion-token corpus). `\citep{soldaini2024dolma}`
  - Nemotron-CC (refined Common Crawl; long-horizon pretraining). `\citep{su2025nemotroncctransformingcommoncrawl}`
  - StarCoder/Stack data for code. `\citep{lozhkov2024starcoder}`
- Midtraining / multi-stage training:
  - continued pretraining in new domains; “cooldown” and targeted upsampling. `\citep{gururangan2020dontStopPretraining}`
  - microannealing as compute-efficient mixture probing (OLMo 2 style). `\citep{olmo20242olmo2furious}`
- Stability:
  - QK-norm as architectural headroom against spikes. `\citep{Dehghani2023ScalingVT}`
  - z-loss as practical stabilizer. `\citep{palm,chameleon,mitch}`
  - skip-step / outlier-step skipping in open training stacks (OLMo lineage). `\citep{olmo20242olmo2furious}`
- Evaluation and contamination:
  - Paloma for corpus fit; formatting sensitivity. `\citep{magnusson2024palomabenchmarkevaluatinglanguage}`
  - prompt-format differences (LM Eval vs OLMes) and contamination threats (Marin’s GSM8K incident). `\citep{marin32bretro}`

## 6.3 Methods and Open Artifacts (`sections/methods.tex`)

### 6.3.1 Open Artifacts (“Model Flow Checklist”)

- What Marin releases per major run:
  - training code and experiment specs
  - explicit mixture manifests (dataset IDs, weights, oversampling)
  - intermediate checkpoints
  - evaluation configs
  - GitHub issues as public experiment log
- Connect to OLMo 2/3 “fully open” rationale.

### 6.3.2 Training Stack and Infrastructure

- Framework: Levanter + JAX; deterministic/resume-friendly design.
- Hardware:
  - 8B: 2x v5e-256 multislice → v4-2048.
  - 32B: preemptible v5p-512 multislices → reserved v4-2048.
- Attention kernels: TPU Splash Attention.
- Precision: params/optimizer fp32; compute bf16.
- Checkpointing policy: permanent full checkpoint every 20k steps (8B retro).

### 6.3.3 Data Sources (Datasets in the Mix)

- DCLM baseline + DCLM HQ. `\citep{dclm}`
- Dolma subsets and Dolmino bundles. `\citep{soldaini2024dolma}`
- Nemotron-CC buckets. `\citep{su2025nemotroncctransformingcommoncrawl}`
- Code corpora: StarCoder data. `\citep{lozhkov2024starcoder}`
- Math corpora and bundles:
  - FineMath 3+ and Dolmino math.
  - MegaMath in 32B Mantis cooldown. `\citep{megamath}`
  - Common Pile StackV2 EDU-filtered Python introduced at ~174k in Mantis. `\citep{commonpile,marin32bretro}`
- Marin-created datasets:
  - markdownified corpora (ArXiv, StackExchange, Wikipedia)
  - Datashop Science QA (generation pipeline described in retro).

### 6.3.4 Architectures

- 8B: Llama-style decoder-only; seq len 4096; Llama 3 tokenizer. `\citep{marin8bretro}`
- 32B:
  - Phase 1: Llama-3-style attention.
  - Phase 3+: Qwen3-style attention with QK-norm (warm-start). `\citep{marin32bretro,qwen3,Dehghani2023ScalingVT}`

### 6.3.5 Optimization and Schedules

- AdamW across runs; weight decay exceptions for embeddings and layer norms (8B retro). `\citep{marin8bretro,loshchilov2017decoupled}`
- WSD-S and WSD:
  - Kestrel uses WSD-S; cycle spacing change at ~step 200k is a key diagnostic.
  - Ocelot switches to WSD and adds EMA monitoring (beta 0.995).
- z-loss as late-regime stabilizer (Spoonbill fix; later made default). `\citep{marin8bretro,palm,chameleon,mitch}`
- AdamC in 32B cooldown decay phases (Bison/Mantis). `\citep{marin32bretro}`

### 6.3.6 Shuffling and Sampling Permutations (Must Be Precise)

This subsection is the “linear vs Feistel” deep dive.

- Goal: within each batch, approximate i.i.d. sampling from the full training distribution to reduce within-batch correlation and avoid long correlated stretches (“phases”). `\citep{marin32bretro}`
- Constraint: must be reproducible, cheap, and stateless (no materialized permutation tables).
- Levanter implements PRP shuffling by permuting dataset indices inside the loader (random-access). `\citep{levanterprp}`

Affine/LCG permutation:

- Definition: choose integers a,b with gcd(a,N)=1 for dataset length N, and map indices by `p(x) = (a*x + b) mod N`. `\citep{marin32bretro,levanterprp}`
- Property: adjacent indices have constant modular distance: `p(x+1) - p(x) ≡ a (mod N)`.
- Why it can fail: if the underlying dataset is block-structured and not pre-shuffled, a constant-stride walk can preserve structure and create non-stationary “phases” in training loss even if mixture weights are stable.

Feistel PRP (Levanter implementation details):

- Concept: split bits into L/R halves and apply several rounds `(L,R) <- (R, L xor F(R,k_i))` to achieve avalanche-like mixing.
- Levanter specifics:
  - embed [0,N) into [0,m) where m is next power-of-two
  - apply bit-level Feistel; for non-power-of-two N use cycle-walking until output in [0,N)
  - default rounds r=5
  - round function on the right half: `F(R,k) = ((R * 2654435761) + k) mod 2^{|R|}`.
- Tie to experiment: Bison cooldown phase shift disappears when switching to Feistel in Mantis.

## 6.4 Experiments and Retrospectives (`sections/experiments.tex`)

### 6.4.1 Evaluation Setup

- Retros use LM Eval Harness defaults; note prompt/template differences across harnesses.
- Report standard suites: MMLU, GSM8K, MATH, HumanEval, BBH, GPQA, MMLU-Pro.
- Track Paloma (esp c4_en) to measure format/domain fit. `\citep{magnusson2024palomabenchmarkevaluatinglanguage}`
- Contamination caveat: benchmark scores are noisy; Marin shows a concrete cached contamination incident.

### 6.4.2 Marin 8B Retrospective

Anchor early with `marin-timeline.png` and the token ranges (Kestrel/Ocelot/Jellyfish/Phoenix/Starling). `\citep{marin8bretro}`

#### 6.4.2.1 Phase 1: Kestrel (0→2.7T tokens; DCLM mix + WSD-S)

- Hardware: 2x v5e-256 multislice.
- Batch: 4Mi tokens/step (1024 x 4096).
- Peak LR: 1e-3; weight decay 0.05; no z-loss initially.
- Mix (normalized). `\citep{marin8bretro,dclm}`

| Dataset | Share |
| --- | ---: |
| DCLM Baseline | 92.6% |
| StarCoder Data | 6.1% |
| ProofPile 2 | 1.3% |

- WSD-S cycle spacing diagnostic (figures listed in Evidence Map):
  - first 200k steps: decay every 10k steps for 1k steps
  - after ~step 200k: decay every 20k steps for 2k steps

#### 6.4.2.2 Phase 2: Ocelot (~2.7T→3.78T tokens; WSD + EMA)

- Hardware: move to v4-2048.
- Batch increases to 12Mi tokens/step at ~2.77T tokens; LR scaled by sqrt(3) to 1.7e-3.
- EMA monitoring introduced (beta 0.995); “EMA gap” shows stable separation in eval loss.
- Operational fix: RoPE settings Llama2 → Llama3; brief spike attributed.

#### 6.4.2.3 Interlude: Microannealing

Microannealing is the “cheap mixture search” loop.

- HQ-only upsampling improves HQ-domain losses but degrades downstream task accuracy.
- Hypothesis: HQ sources lack few-shot/task-like formats.
- FLAN in moderation counteracts this; best reported: 70% PT / 15% FLAN / 15% HQ.
- FLAN alone at 30% underperforms baseline.

#### 6.4.2.4 Phase 3: Jellyfish (3.78T→4.78T tokens; first cooldown)

- LR decay: 1.7e-3 → 1.7e-4 over 1e12 tokens (79,500 steps at 12Mi tokens/step).
- Mix (normalized). `\citep{marin8bretro}`

| Dataset | Share |
| --- | ---: |
| Dolmino DCLM HQ | 67.8% |
| Dolma peS2o | 10.8% |
| FineMath 3+ | 6.3% |
| Dolma ArXiv | 5.2% |
| Dolma StackExchange | 3.2% |
| StarCoder | 2.2% |
| Dolma Algebraic Stack | 2.1% |
| Dolma Open Web Math | 0.9% |
| Dolma Megawika | 0.8% |
| Dolma Wikipedia | 0.7% |

- Key interpretation notes:
  - Paloma c4_en perplexity increases (format mismatch hypothesis).
  - the mix excludes FLAN at this point (retro notes suspicion about templating).

#### 6.4.2.5 Phase 4: Phoenix (4.78T→11.1T tokens; reheat + Nemotron-CC transition)

- Transition:
  - 2,000-step interpolation (~25.2B tokens)
  - LR rewarm linearly from 1.7e-4 → 1.7e-3 over same window; then hold.
- Evidence: `8b-phoenix-transition.png` shows a brief spike and recovery to slightly lower loss.

#### 6.4.2.6 Deeper cooldowns (Raccoon + Spoonbill)

- Raccoon:
  - cooldown 1.7e-4 → 1.7e-5 over ~100B tokens
  - anomaly: loss creeps upward (figure `8b-raccoon-loss-increase.png`)
- Spoonbill:
  - cooldown to 3e-5
  - add Tulu v3 data (0.3%) + FLAN (1%)
  - anomaly repeats (figure `marin-8b-spoonbill-loss.png`)
  - diagnosis: lm_head norm explosion (figure `8b-spoonbill-norms.png`)
  - fix: z-loss 1e-4 (figure `8b-spoonbill-zloss.png`)

#### 6.4.2.7 Phase 5: Starling (second cooldown; 1.34T-token cooldown)

- Changes vs Jellyfish:
  - deeper cooldown to 1.7e-5
  - z-loss 1e-4
  - batch increases to 16Mi
- Cooldown length: retro explicitly states “1.34T tokens” (timeline boundaries are approximate).
- Mix (proportion + oversampling). `\citep{marin8bretro}`

| Dataset | Proportion | Oversampling |
| --- | ---: | ---: |
| Nemotron CC Medium | 22.1% | 1x |
| Nemotron CC HQ Synth | 17.8% | 1x |
| Nemotron CC Medium Low | 10.1% | 1x |
| Nemotron CC HQ Actual | 6.0% | 1x |
| Nemotron CC Medium High | 5.4% | 1x |
| Nemotron CC Low Actual | 4.6% | 1x |
| Nemotron CC Low Synth | 4.1% | 1x |
| Marin ArXiv Markdown | 5.2% | 5x |
| Dolmino peS2o | 5.2% | 5x |
| StarCoder Data | 4.5% | 1x |
| ProofPile 2 | 4.5% | 1x |
| FineMath 3+ | 3.0% | 5x |
| Dolmino FLAN | 3.0% | 10x |
| Dolmino StackExchange | 1.5% | 5x |
| Marin StackExchange Markdown | 1.5% | 5x |
| Dolmino Math | 0.8% | 10x |
| Marin Wikipedia Markdown | 0.3% | 5x |
| Dolmino Wiki | 0.3% | 5x |
| Marin Datashop Science QA | 0.1% | 5x |

- Notes to include in paper:
  - “Dolmino Math” is a bundle (GSM8K, MetaMath, SynthMath, MathCoder2 synthetic, TinyGSM-MIND, etc.); this becomes relevant in 32B Bison contamination.
  - Datashop Science QA pipeline: annotate small subset with LLM → train quality filter → score/select at scale.
  - Markdown corpora motivation: preserve structure (tables/LaTeX) and match downstream markdown-heavy usage.

#### 6.4.2.8 8B Results and SFT Probe

- Base benchmark table (Deeper Starling vs Llama 3.1 8B vs OLMo 2 7B). `\citep{marin8bretro}`
- SFT probe:
  - datasets listed in retro
  - 5Gi tokens, batch 512Ki, LR 1.7e-4
  - trade-off: instruction metrics improve; some base metrics degrade; connect to OLMo 2 similar observation. `\citep{marin8bretro,olmo20242olmo2furious}`

### 6.4.3 Marin 32B Retrospective

#### 6.4.3.1 Pretraining Mix and Batch Schedule

Pretraining mixture (normalized share). `\citep{marin32bretro}`

| Dataset | Share |
| --- | ---: |
| nemotron_cc/medium | 30.69% |
| nemotron_cc/hq_synth | 24.70% |
| nemotron_cc/medium_low | 13.98% |
| nemotron_cc/hq_actual | 8.30% |
| nemotron_cc/medium_high | 7.49% |
| nemotron_cc/low_actual | 6.37% |
| nemotron_cc/low_synth | 5.70% |
| starcoderdata | 2.27% |
| proofpile_2 | 0.50% |

Batch schedule (hardware-driven divisibility). `\citep{marin32bretro}`

| Start step | Global batch | Tokens/batch |
| ---: | ---: | ---: |
| 0 | 8192 | 32Mi |
| 18,500 | 7680 | 30Mi |
| 21,010 | 8192 | 32Mi |

#### 6.4.3.2 Phase 1: Baseline scale-up and loss spikes

- Tokens: ~2.679T (80k steps, seq len 4096).
- Baseline optimizer (retro): AdamW, peak LR 7e-4, WSD-style warmup/hold/decay (warmup 1%, decay 40%), WD 0.05, EMA 0.995, z-loss 1e-4. `\citep{marin32bretro}`
- Evidence figures: `32b-spiky-loss.png`, `32b-update-spike-precede-loss-spike.png`, `32b-loss-comparisons.png`.
- Mitigations (with precise step numbers from retro):
  - tighten max grad norm from 1.0 → 0.2 at ~56.4k
  - add clip update norm at 72,233 (rolling 128, 2-sigma), briefly off ~74k–80k
  - add skip bad steps (2-sigma, rolling 128)
- Issue #1368 details to explicitly include:
  - update spikes precede loss spikes; gradient norms typically spike after
  - update spikes larger in lower layers
  - during update spikes: Adam first moment grows ~2x while second moment mostly unchanged
  - hypothesis: consecutive aligned gradients increase momentum → big update. `\citep{marinissue1368}`
- Issue #1390 operational failure:
  - update clipping inadvertently turned off
  - a small spike fails to recover; run “cooked.” `\citep{marinissue1390}`

#### 6.4.3.3 Phase 2: Recovery attempts (discarded)

- Necromancy restarts (figure `32b-bad-spike.png`).
- Optimizer swap: Muon
  - temporary relief (figure `32b-muon-vs-adam.png`)
  - divergence (figure `32b-muon-can-into-space.png`).

#### 6.4.3.4 Phase 3: QK-norm switch

- Switch: warm-start from step 80k Llama 32B into Qwen3-style attention with QK-norm.
- Evidence: one-time loss penalty recovers within ~10B tokens; spikes disappear (figure `qk-recovery.png`). `\citep{marin32bretro}`

#### 6.4.3.5 Phase 4: Cooldowns (Bison vs Mantis) and data-system failures

Both cooldown attempts: 32k steps (160k→192k), seq len 4096, global batch 8192, ~1.074T tokens. `\citep{marin32bretro}`

Bison cooldown:

- Mixture is “Starling-like” (70/30 Nemotron/HQ); within HQ, FLAN upsampled 10x and Dolmino math 2x.
- Schedule: no warmup; linear decay over 160k→192k; AdamC enabled during decay; z-loss throughout.
- Mix table (normalized share). `\citep{marin32bretro}`

| Dataset | Share |
| --- | ---: |
| nemotron_cc/medium | 21.49% |
| nemotron_cc/hq_synth | 17.29% |
| nemotron_cc/medium_low | 9.79% |
| nemotron_cc/hq_actual | 5.81% |
| nemotron_cc/medium_high | 5.24% |
| nemotron_cc/low_actual | 4.46% |
| nemotron_cc/low_synth | 3.99% |
| arxiv_markdownified | 7.41% |
| dolmino/pes2o | 7.41% |
| finemath-3-plus | 4.33% |
| dolmino/flan | 4.33% |
| stackexchange_custom | 2.18% |
| dolmino/stackexchange | 2.18% |
| starcoderdata | 1.59% |
| all_math | 1.08% |
| proofpile_2 | 0.35% |
| wikipedia_markdown | 0.47% |
| dolmino/wiki | 0.47% |
| medu_science_qa | 0.15% |

- Failure 1: GSM8K contamination via cached Dolmino math bundle:
  - Dolmino math included GSM8K test in `test.json`
  - preprocessing later dropped `test.json`, but cached dataset persisted
  - contamination harmed LM Eval prompt performance because contaminated data used OLMes formatting (prompt-format mismatch yields high surprisal on structured tags). `\citep{marin32bretro}`
- Failure 2: shuffling anomaly / phase shift:
  - near ~190k steps, training loss phase-shifts while eval remains stable (figure `32b-shuffle-spike.png`)
  - interpreted as data-ordering artifact from linear PRP (co-prime step size; unlucky stride)
  - note: per-component mixture sampling keeps proportions stable, but did not prevent correlated batches. `\citep{marin32bretro}`

Mantis cooldown:

- Restart from 160k; keep optimizer schedule identical.
- Change 1: switch sampling permutation from linear PRP to Feistel PRP.
- Change 2: cleaner math mix:
  - replace Dolmino math with MegaMath splits
  - later add Common Pile EDU-filtered Python around ~174k; renormalize HQ budget. `\citep{marin32bretro,megamath,commonpile}`
- Mix table (normalized share). `\citep{marin32bretro}`

| Dataset | Share |
| --- | ---: |
| nemotron_cc/medium | 21.49% |
| nemotron_cc/hq_synth | 17.29% |
| nemotron_cc/medium_low | 9.79% |
| nemotron_cc/hq_actual | 5.81% |
| nemotron_cc/medium_high | 5.24% |
| nemotron_cc/low_actual | 4.46% |
| nemotron_cc/low_synth | 3.99% |
| megamath/web | 5.57% |
| arxiv_markdownified | 4.54% |
| megamath/text_code_block | 4.24% |
| dolmino/pes2o | 4.54% |
| megamath/web_pro | 1.27% |
| megamath/translated_code | 0.61% |
| megamath/qa | 0.59% |
| finemath-3-plus | 2.66% |
| dolmino/flan | 2.66% |
| stackexchange_custom | 1.34% |
| dolmino/stackexchange | 1.34% |
| starcoderdata | 1.59% |
| proofpile_2 | 0.35% |
| wikipedia_markdown | 0.29% |
| dolmino/wiki | 0.29% |
| medu_science_qa | 0.09% |

- Evidence:
  - shuffle phase shift removed (`32b-feistel-vs-lcg.png`)
  - Paloma c4_en + Paloma avg improve (`32b-paloma-c4-en-permutation.png`, `32b-paloma-average-permutation.png`).

#### 6.4.3.6 32B Results

- Present Bison vs Mantis results table and compare to OLMo 2 32B Base and Qwen 2.5 32B Base.
- Tie improvements to the specific fixes: contamination removal, shuffle mixing improvement, math mix changes.

### 6.4.4 Open Development and Experiment History

- Marin has an LLM-generated summary page grouping GitHub issues; it explicitly warns “do not trust without verification.” `\citep{marinsummary}`
- Use summary page as an index, but cite primary issue threads for any factual claims.
- Candidate issues to curate (examples from summary page): tokenizer selection (issue #524), HTML→text conversion, compression-ratio filtering, LR schedule exploration, quality classifier experiments.

## 6.5 Conclusion (`sections/conclusion.tex`)

- Distill transferable lessons:
  - mid-flight adaptation works, but requires frequent diagnostics
  - microannealing is a practical mixture-search proxy
  - late-regime stability is a real failure mode (z-loss as guardrail)
  - 32B stability required architectural change (QK-norm)
  - shuffle quality matters even given perfect per-domain mixture proportions
  - cache hygiene is part of evaluation integrity
- Limitations:
  - benchmark contamination uncertainty
  - English+code bias and dataset constraints
  - limited post-training exploration in 32B paper

## 7. Appendices Plan (Optional)

- Full hyperparameter tables per phase (8B + 32B).
- Dataset manifest appendix (IDs, weights, oversampling rules).
- Shuffling appendix (pseudo-code for affine PRP and Feistel PRP; cycle-walking details).
- Contamination appendix (cache invalidation strategies; prompt-format mismatch checklist).

## 8. Paper-Writing TODOs

- Decide whether to add an explicit citation for Feistel networks (cryptography) vs relying on the implementation citation (`levanterprp`).
- If expanding “open development” section: add bib entries for a small number of high-signal issues (tokenizer selection #524 is especially relevant).
