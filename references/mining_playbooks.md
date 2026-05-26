# Mining Playbooks (Reference)

> Companion to SKILL.md §Step 4 deep mode. SKILL.md is the entry; this file provides the full
> **domain detection + universal mining + domain-specific mining** checklist.

Different papers hide depth in different places:
- A theory paper hides depth in assumption tightness and proof step intuitions
- An ML systems paper hides it in tensor shapes and training recipe details
- A wet-lab paper hides it in control groups and statistical assumptions
- A field study hides it in sampling protocol and ecological validity

**Do NOT apply a fixed checklist to every paper.** Use the 4-step framework below:

---

## Step A — Identify the paper's domain(s)

A single paper can belong to multiple domains (e.g., a deep learning method paper is often both "neural network method" + "new training recipe" + "iterative algorithm"). Tick all that apply:

- [ ] **Neural network / deep learning method** — introduces a new network architecture, module, or backbone
- [ ] **New loss / regularizer / training objective** — introduces a new loss function or regularization
- [ ] **Training recipe / hyperparameter study** — main contribution is training details or hyperparameter choices
- [ ] **Math / theory / proof-heavy** — main contribution is a theorem, proof, or bound
- [ ] **Algorithm with iterations / dynamics** — iterative algorithm (optimization, MCMC, gradient flow, etc.)
- [ ] **Empirical study (wet lab / human subjects / field measurement)** — main contribution is empirical findings
- [ ] **Systems / performance / scaling** — focus on latency / throughput / scaling laws
- [ ] **Dataset / benchmark construction** — introduces a new dataset or evaluation protocol
- [ ] **Survey / position paper** — survey or position paper
- [ ] **Application paper** — known method applied to a new domain
- [ ] **Critique / replication paper** — replicates or challenges prior work

---

## Step B — Universal mining (always applies)

Regardless of domain, walk these 10 items:

- [ ] **Implicit prerequisites** — concepts the paper assumes the reader knows but doesn't define (math tools, building blocks, prior frameworks, domain conventions). Adding the term to a glossary is **not enough** — for each prerequisite, surface **all three**:
  1. **Source** — where does this object come from in the paper / network / pipeline / proof? Materialize it: shape, type, role, how it was constructed. ("M is the per-head attention weight at layer ℓ, ℝ^(D×D), formed by softmax(QK^T/√d).")
  2. **Use** — what is the paper actually doing with this object? Which property is being extracted? ("They want to measure M's rank concentration → does this head route through few or many tokens?")
  3. **Lego toy** — a 2×2 or 3×3 worked instance the reader can compute mentally. Abstract definitions without a toy leave the reader with a phrase but no picture.

  This is the mining-side raw material for `teaching_moves.md` #10 toolchain unfurling. Surface during the mining pass; instantiate fully (Steps A–D) when you teach the tool downstream.
- [ ] **Hidden cost / complexity** — which of {compute, memory, data, time, manual labor, hardware, sample collection} is the silent bottleneck?
- [ ] **Confidence vs point estimate** — which claims are reported with CIs / error bars / replicates / sensitivity sweeps, and which are single numbers?
- [ ] **Design decision audit** — Every paper makes a sequence of load-bearing design choices (loss form, regularizer family, architecture cardinality, data-split rule, sampling protocol, proof strategy, chosen estimator). For each, answer **all five**:
  1. What alternative(s) did the authors consider, or could a reasonable reader propose? (Name 2–3 concretely.)
  2. **What specifically goes wrong with each alternative** — not "it's worse" but "it does X bad thing through mechanism Y at scale Z".
  3. Is there a number that shows the gap? (an ablation row, a comparison table, a closed-form derivation, an entropy/variance/error count.)
  4. Did the paper itself justify this choice in prose, or did it slide past with "we use X"? Flag the slides.
  5. What common misconception would a reader bring to this choice that should be defused first? (e.g., "I assumed the latent dim was task-dependent" → no, it's fixed.)

  This audit is the **mining-side raw material** for `teaching_moves.md` #9 motivation walkthrough. Aim for 2–4 audited design choices per deep note — those are the ones that get a dedicated `## Why X?` chapter.
- [ ] **Load-bearing parameter** — every method has one hyperparameter / assumption / dataset choice that mostly determines the outcome. Find it.
- [ ] **What would break it** — counterfactuals, regime shifts, edge cases the method silently can't handle.
- [ ] **Paper-vs-reality gaps** — does the official repo / supplementary / replication actually match the text? Common gaps: hyperparam typos, missing details, different defaults.
- [ ] **Sections that rush relative to importance** — a 1-paragraph section on a critical mechanism is a red flag worth expanding.
- [ ] **Worked example with concrete numbers** — for at least 3 non-trivial claims, replay with toy values the reader can verify mentally.
- [ ] **"But what about…?" sub-question for every major section** — at the end of each, ask "but what about X edge case?" Even if the answer is "not handled — this is a limitation".

---

## Step C — Domain-specific mining (load only what Step A ticked)

Load **only** the rows whose domain you ticked in Step A. **Skip the others** (don't pad with patterns that don't fit):

| Paper domain | Add these mining patterns |
|---|---|
| **Neural network method** | Tensor shapes `(B, T, D)` annotated on every interface; pseudocode for forward / backward; padding / masking for variable-length input; module-by-module parameter count breakdown |
| **New loss / regularizer** | Show loss at degenerate inputs (collapsed / random / noise); walk gradient flow; show what each term in the sum is responsible for |
| **Training recipe** | Each non-default hyperparameter's source (text / appendix / repo config); compare to backbone defaults; document any "hidden" hyperparameter that the paper text doesn't emphasize |
| **Math / theory / proof** | Where each assumption is used; counter-example for each "obviously"; what relaxation breaks the bound; intuition for why this norm / space / topology; trace the proof's load-bearing inequality |
| **Algorithm with iterations** | ONE full iteration with concrete numbers; convergence behavior under different inits; failure / divergence cases |
| **Empirical study (wet lab / human / field)** | Sample size adequacy; controls / confounds; statistical test assumptions (normality / independence); ecological validity vs lab artifact |
| **Systems / performance** | Latency / throughput / memory profile; scaling regime (where the bottleneck shifts); hardware specifics |
| **Dataset / benchmark** | How items were sampled; label / annotation process; train/test leakage check; difficulty distribution; representativeness |
| **Survey / position paper** | Coverage criteria; selection bias in citations; alternative framings the author rejected; what's NOT in the survey but should be |
| **Application paper** | How well does the borrowed method fit this new domain? What domain-specific failure modes appear that the original didn't have? |
| **Critique / replication** | What's the original's strongest defense? What does this replication NOT test? |

---

## Step D — Express form

For any non-trivial algorithm / computation / inference uncovered above, instantiate as one of:

- 5–20 lines of pseudocode (ML: with tensor shape comments; theory: numbered proof steps; empirical: numbered protocol steps)
- A figure (architecture / flow / data / phenomenon / ablation)
- A worked-example table

Form depends on domain — don't force ML form on a theory paper or vice versa.

---

## Output rule

After Step A, **silently apply the right playbooks**. **Do NOT write a visible "Step A domain identification" or "Loaded playbooks: X, Y" section into the reader output** — that is your private planning. The reader doesn't care which playbooks you loaded; they want to learn the paper.

If you want to be transparent about your reading choices in a reader-facing way, use Appendix A (Paper figure / table audit) in the deep_note template — that's about which paper figures you deep-read vs skipped, which is useful to readers.
