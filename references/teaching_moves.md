# Teaching Moves (Reference)

> Companion to SKILL.md §Step 4 deep mode. Mining tells you **what to dig for**;
> teaching moves tell you **how to present**. These 8 moves are domain-agnostic — they
> are what distinguishes a tutor from a summarizer.

A good tutor uses these moves regardless of paper type. Apply the ones that fit.

---

## 1. Anchor concept first

**Definition**: In the preface, name the ONE foundational fact the paper rests on (a geometric reality / a statistical inevitability / an algorithmic identity / a physical constraint). State it with a short visual or worked example. Then add a "where this appears" table showing how this anchor reappears across §2 / §3 / §4 / etc.

**Examples across domains**:
- A theory paper might anchor on "all my bounds rely on Lipschitz continuity of $f$" — then track where the Lipschitz assumption tightens in each lemma.
- An ML paper might anchor on "the encoder's latent representations live on a low-dimensional manifold, not a high-dim isotropic ball" — then track where this affects loss design / regularizers / collapse.
- A wet-lab paper might anchor on "Drug A binds receptor R selectively at sub-µM affinity" — then track where selectivity appears in dose-response / off-target / clinical relevance sections.

**Why it works**: Without an anchor, the deep reading reads as a list of unrelated chapters. With one, every section connects back to a single load-bearing fact, which is how experts actually organize their understanding.

**Don't**: Force an anchor if the paper has none. Better to declare 2–3 anchors than to fabricate one.

---

## 2. Lego-level toy before paper-scale

**Definition**: Every worked example uses minimal toy values **first**, then says "the paper actually uses ${big numbers}".

**Examples**:
- An iterative algorithm with 3000 samples → first show 5 samples on a 1D function
- A 192-dimensional latent → first show a 2-dimensional latent with hand-pickable distance computations
- A 12-layer transformer → first show 2-layer toy attention with a 3-token input
- 18,500 trajectories → first show 3 trajectories of 5 frames each
- A statistical test on n=10,000 subjects → first show the same test on n=4 with hand-pickable values

**Template**: "For intuition, let's use a toy: ${small numbers}. The paper actually uses ${big numbers}, but the underlying logic is identical."

**Don't**: Drop paper-scale numbers on the reader without a toy first — their brain shuts down.

### Important: toys are encouraged, NOT discouraged

The Evidence Provenance Rule (figure_contract.md §1) introduces `[synthetic-demo]` and `[interpretation]` tags. These rules exist to let toy demos coexist with real evidence **without confusion**, not to reduce toys. Toys + lego examples are the tutor's core tool.

A `deep+fig=on` note should have **≥3 `[synthetic-demo]` figures** (per figure_contract.md §2 functional budget). That's a floor, not a ceiling.

---

## 3. Toy demo to SEE the phenomenon

**Definition**: For any abstract claim (representation collapse, error accumulation, sweet spot, mode collapse, scaling break, phase transition), build a minimal runnable visual that **demonstrates the phenomenon** BEFORE introducing the formal method that solves it. The reader should see "yes, that's a real problem" before reading "here's the fix".

**Examples across domains**:
- For "autoregressive rollouts accumulate error" → 5–20 lines of numpy simulating 30 rollouts of a 1D process with small per-step noise, plotting how the spread blows up
- For "this estimator is biased when n is small" → toy simulation with n=10 vs n=10,000, showing the bias gap
- For "training collapses without this regularizer" → 2D toy with regularizer on/off, showing latent points converging to a single point in the unregularized run

**Implementation**: pseudocode + matplotlib figure with seeded randomness so the figure is reproducible.

**Don't**: Jump straight to the method without showing the problem exists.

---

## 4. Thought experiment with extreme values

**Definition**: For the paper's load-bearing parameters, walk the reader through what happens at extreme settings. This makes the load-bearing nature explicit.

**Examples**:
- "Imagine if dimension = 1, then ..." → shows the lower-bound failure mode
- "Imagine if horizon → ∞, then ..." → shows the upper-bound failure mode
- "Imagine if batch_size = 1, then ..." → shows batch-statistical losses break
- "Imagine if assumption A doesn't hold, then the bound becomes ..." → shows assumption load

**Template**: "Suppose ${extreme value}: ${failure mode that would emerge}. This is why ${parameter} must stay in ${normal range}."

**Typically combines with**: counter-intuitive dialogue (#5).

---

## 5. Counter-intuitive explicit dialogue

**Definition**: For counter-intuitive points, don't just state the conclusion. Walk through naive → why wrong → actual in three acts.

**Template**:
> ### Counter-intuitive N: "You might think X"
> **You might think**: ${naive view}
> **But actually**: ${counter-intuitive truth}
> **Why naive is wrong**: ${mechanism explaining the gap}
> **Worked example showing the gap**: ${concrete computation, ideally with numbers}

**Examples across domains**:
- A common ML one: "You might think higher capacity = better, but actually there's a U-shape because ${reason}, e.g. at ${value}, performance drops to ${number}."
- A common stats one: "You might think p < 0.05 means the effect is real, but actually with multiple testing the false discovery rate is ${number}."
- A common theory one: "You might think this bound is tight, but actually it loses a factor of $\log n$ here because ${reason}."

**Why it works**: Naive views aren't dumb — they're the natural extrapolation from related concepts. Acknowledging the naive view, then explaining why the actual answer differs, is more sticky than just stating the truth.

---

## 6. Claim → deliver immediately

**Definition**: After stating a claim, deliver the evidence **in the same paragraph or figure**. Don't forward-reference.

**Forbidden patterns**:
- "We'll see in §5 that hierarchy improves performance" → forces reader to jump
- "Will be explained below" → flow break
- "See Figure 12" when figure 12 is 500 lines away

**Good patterns**:
- "Method X improves the long-horizon success rate from 17% to 61% on the hardest setting (paper Tab 2)" → numbers in-line
- "As shown ↓" → followed immediately by the figure

**Heuristic for self-diagnosis**: if you wrote "we'll see in §N" or "more on this below", it's an 80% smell of forward-reference. Inline the evidence.

---

## 7. Multi-view of the same concept

**Definition**: Show the same core concept from multiple angles, so readers with different thinking styles can hook in.

**Example multi-view set for a single complex concept**:
- View A: scatter plot in 2D (geometric intuition)
- View B: variance spectrum on log scale (statistical intuition)
- View C: numerical summary table (tabular intuition)
- View D: multiple camera angles of the same point cloud (perspective)

**When to use**: The paper's 1–3 most important claims deserve multi-view. Secondary claims, single figure is enough.

**Don't**: Draw 5 nearly identical figures to pad. Multi-view requires **complementary dimensions** (geometric / statistical / tabular / perspective / time-series / etc.), not repetition.

---

## 8. Real-data evidence > illustrative simulation

**Definition**: If the paper's checkpoint / dataset is publicly accessible, **load it** and visualize actual model behavior or actual data. If it's not accessible, **explicitly mark figures as illustrative**.

**Examples of real-data evidence**:
- Load the released checkpoint, run inference on standard inputs, plot real activation patterns / latent embeddings / prediction errors
- Download the released dataset, plot real distributions / class balance / typical samples
- Pull the released numerical results table, replicate the headline plot exactly

**Why it's higher trust**: Readers know you're not picking numbers that make the story look good.

**If not accessible** (no public weights / requires hardware you don't have):
- Explicitly tag figure captions as "illustrative reproduction — actual ckpt not accessible"
- Explain in prose why you can't measure for real
- Use hedge words like "illustrative" instead of "actual / measured"

**Core principle**: this is part of the **Evidence Provenance Rule** (figure_contract.md §1). Tag honestly, then toy freely.

---

## 9. Motivation walkthrough (dedicated chapter for load-bearing design choices)

**Definition**: For the paper's 2–4 most load-bearing design choices (the regularizer form, the loss family, the data split rule, the architecture cardinality, the sampling protocol, the proof strategy, the chosen estimator, etc.), **don't bury "why this" inside method prose**. Lift it into a dedicated motivation walkthrough that sits **before** the corresponding method exposition. Readers need to know *why this shape* before they read *how it works*.

Apply at least **4 of the 7 moves below per motivation**. A deep note must include **≥2 full motivation walkthroughs** (one per top design choice).

### The 7 motivation moves

1. **Pose "Why X?" as a section heading**, not as an aside.
   - Good: `## 4.2 Why isotropic Gaussian? Three reasons`
   - Bad: a passing sentence "we use isotropic Gaussian, which is convenient" in the middle of method prose.

2. **Multi-angle numbered breakdown** when there are several reasons. ① max-entropy → ② algebraic ease → ③ checkability. Each reason gets its own subsection with intuition + formula + concrete payoff. Numbering tells the reader "this list is closed — you've seen all the reasons".

3. **Defuse a common misconception first**. Name the wrong picture readers usually bring before giving the motivation.
   - Example: "You might think the latent dimension is task-dependent, but it's fixed at 192 across all tasks — which is exactly why the next argument matters."
   - This pulls every reader to the same starting line.

4. **Reverse argument with concrete mechanism**. Don't say "the alternative is worse" — say "the alternative does X bad thing through mechanism Y at scale Z".
   - Vague: "full-space Gaussian wastes capacity."
   - Mechanism: "with 4 real-signal dims out of 192 ambient, full-space Gaussian forces the model to manufacture variance in 188 zero-signal directions, which dilutes the signal-to-noise of latent-distance metrics used in planning."
   - The bar is: a reader can quote one specific mechanism the alternative breaks on.

5. **3-panel "alternative vs ours" figure**: (left) ground-truth structure, (middle) what the baseline does to it, (right) what the paper does. Reader sees the gap in one glance. Save as a `[synthetic-demo]` or `[interpretation]` figure depending on whether you used real or toy data.

6. **Comparison number table** — collapse the abstract argument into a small table where the chosen option visibly wins.
   ```
   | Option   | Entropy (at σ²=1) |
   |----------|-------------------|
   | **Gaussian** | **1.419 ← max**   |
   | Triangle | 1.396             |
   | Laplace  | 1.322             |
   | Uniform  | 1.237             |
   ```
   One number per row; mark the chosen row. This turns "we pick X because of reason Y" into "X is the row that scores best — here's the table".

7. **Self-rebuttal at the chapter end** — pose the sharpest objection a skeptical reader would raise, then honestly answer it.
   - Template:
     > **Common sharp objection**: "${objection a hostile reader would raise}"
     > **Honest answer**: ${not-defensive answer — concede what's true ("partly correct"), then explain why the design still holds ("the strength is diluted because …")}.
   - Closing a motivation with self-rebuttal converts "I read the argument" into "I trust the argument" — because the reader sees you already thought of their best counter and didn't dodge it.

### Output discipline (deep mode)

- **≥2 motivation walkthroughs** per deep note, one per top design choice.
- Each walkthrough uses **≥4 of the 7 moves** (moves #1 and #4 are highly recommended; the rest are optional combinations).
- Each walkthrough sits **before** the method exposition it motivates, not after.
- Tag every claim inside the walkthrough with provenance — `[paper-reported]` if the paper actually argued it, `[interpretation]` if you reconstructed the argument, `[synthetic-demo]` for the comparison numbers/figures you produced.

### The "circularity test" (failure mode to avoid)

The most common failure: writing motivation that is really a one-line restatement of the method's name.

- **Bad (circular)**: "We use SIGReg because we want isotropic Gaussian latents." → could fit any regularizer; says nothing.
- **Good (non-circular)**: "We need isotropic Gaussian because (a) it maximizes latent entropy under variance constraint, (b) its 1-D projections are still Gaussian which lets us check it cheaply via Cramér-Wold, (c) the alternative — anisotropic Gaussian — requires estimating an N×N covariance which is statistically expensive at the batch sizes we train with." → contains specific facts the reader couldn't have guessed.

**The test**: read your motivation paragraph aloud after replacing the method name with `X`. If it still parses as a generic argument, it's circular — rewrite with concrete mechanisms.

---

## 10. Toolchain unfurling (for every math tool or prerequisite concept introduced)

**Definition**: When you introduce a math tool (SVD, Fisher information, KL divergence, Cramér distance, mutual information, etc.) or a prerequisite construct (a matrix that comes from somewhere in the network, an estimator, a sample statistic), **do not drop it as a fait accompli**. Unfurl it along the dependency chain — the reader must see *what* the object is, *why* you're touching it, and *why this tool* before you write a single formula about it.

### The failure mode this fights

The most common skip: introducing a tool with one line — e.g., *"any matrix M has an SVD M = UΣV^T, and the singular values σ_1 ≥ σ_2 ≥ ... are its fingerprint"* — which leaves the reader stuck on three questions the agent never answered:

1. **What is M?** Is it an attention weight matrix? a Jacobian? a covariance? where does it live in the network / pipeline / proof?
2. **Why am I touching M at all?** What property of M does the paper want to measure — rank concentration? energy distribution? alignment with some subspace?
3. **Why SVD specifically?** (vs eigendecomposition / vs QR / vs polar) — what does each give that SVD does not, and why is SVD the right one for this property?

If any of these three is unanswered, the reader walks away with a phrase ("singular values are the fingerprint") but no model of when or why to actually use it.

### The 4-step toolchain template

Use this whenever §1 (background), §2.0 (motivation), or §2.X (components) introduces a math tool or building block:

- **A. Materialize the object.** What is it concretely? Where in the paper / network / pipeline does it live? Give shape, type, role, and how it was constructed.
  - *Example*: "M ∈ ℝ^(D×D) is the per-head attention weight matrix at layer ℓ, formed by softmax(QK^T / √d). One M per (layer, head) pair; the paper analyzes ~96 of them."

- **B. State what you want from it.** What property are you trying to measure, and why does that property matter to the paper's argument?
  - *Example*: "We want to know how concentrated M is — does this head route information through a few tokens (low effective rank, M ≈ rank-1) or spread it across many (high effective rank)? The paper's claim about 'attention specialization' lives or dies on this question."

- **C. Introduce the tool + a one-paragraph refresher + why-this-tool.** Even if the reader has seen the tool before, give a 2–3 sentence refresher of what it does and what its outputs mean; then justify *why this tool* over the obvious alternatives.
  - *Example*: "SVD factors any real M = UΣV^T with Σ a diagonal of non-negative σ_i in decreasing order; the σ_i tell you exactly how much energy each rank-1 component contributes. We use SVD instead of EVD because M is not symmetric here — only square symmetric matrices have a real EVD with orthogonal eigenvectors. The σ_i sequence is the right answer to step B because effective rank is defined directly from the energy distribution."

- **D. Compute the tool on a 2×2 (or 3×3) toy** with actual numbers so the reader sees what the abstract output looks like.
  - *Example*: "Toy: M = [[2, 0], [0, 0.1]] → σ_1 = 2, σ_2 = 0.1. The spectrum [2, 0.1] tells us at a glance: 95% of M's energy is in one direction → M is effectively rank-1. Now when the paper says 'layer 8 head 3 has σ_1 / σ_2 = 47', you have a mental anchor for what that ratio means."

### Output discipline

- Every math tool introduction in a deep note must complete A→D **before** the tool gets used downstream. No "we apply SVD to M" appearing without the chain done first.
- **Step D (toy computation) is mandatory even for tools the reader 'should' know** — it gives the reader a concrete mental image to map back to when paper-scale instances appear later. This is the single most common skip.
- For very common tools (softmax, dot product), one sentence can cover A+C; but for any tool the paper's main argument **hinges on**, give the full A→D treatment.
- Tag every claim inside the unfurling — `[paper-reported]` if the paper itself stated it, `[interpretation]` if you reconstructed the framing, `[synthetic-demo]` for the toy in step D.

### Self-check: the "fait accompli" test

After writing any tool introduction, read it as if you've never seen this object before. If you find yourself wanting to ask "wait, what *is* M?" or "wait, *why* this tool?" — step A, B, or C is missing. Fix it before the tool gets used.

---

## 11. End-to-end running example through the whole pipeline

**Definition**: When the paper has an end-to-end flow — a training pipeline, an inference pipeline, an iterative algorithm, a multi-step protocol, a multi-lemma proof chain — declare **one running example** at the pipeline entry and carry the **same concrete numbers** through every stage. The reader sees what each stage does to the same data, in sequence, with no jumps and no inventing-their-own-numbers between stages.

### Why local toys are not enough

A deep note typically has many local toys: one per component (move #2 Lego), one per phenomenon (move #3 toy demo), one per math tool (move #10 step D). These are necessary. They are **not** sufficient.

When the reader reaches the pipeline exposition (§3 training / §4 inference / a proof chain / a protocol walkthrough), they don't have a mental anchor tying the components together — because each local toy used different numbers. A running example fills the gap: the reader sees the **same** `z_0 = (0.7, 0.3)` enter the encoder in stage 1, get transformed by the predictor in stage 2, scored by the planner in stage 3, and produce the action in stage 4 — all with numbers they can verify by hand.

### The 4-part template

1. **Declare at the pipeline entry.** Before the first stage, set the example up explicitly with a callout:
   > **Running example (used through §X.1 to §X.N).** Input X = ${concrete value}, hyperparameters = ${concrete values}, batch size = ${small number}, horizon = ${small number}.

2. **Pass through every stage with the same data.** Each stage shows:
   - **Input** from previous stage (with the running-example value)
   - **Operation** applied (the formula, abbreviated if already explained)
   - **Output** (with the running-example value)
   - **One-line interpretation** of what changed and why

3. **Single flow figure.** One matplotlib figure (or mermaid for trivial pipelines) showing every stage as a box, with the running-example value annotated **at each step**. The reader returns to this figure throughout the chapter.

4. **Closing summary.** At the pipeline end:
   > You just saw a complete ${pipeline name} with concrete numbers. The paper actually uses ${paper-scale numbers} — but every step is mechanically identical.

### Where to place it (by paper type)

| Paper type | Where the running example lives |
|---|---|
| Inference / execution pipeline | §4 (Inference / execution details) — declare at §4.0, carry through §4.1–§4.N |
| Training pipeline | §3 (Training details) — declare at §3.0, carry through §3.1–§3.N |
| Iterative algorithm (optimization, MCMC, gradient flow) | §2 main exposition + a dedicated "one full iteration with concrete numbers" subsection |
| Multi-step protocol (wet lab, statistical analysis) | §3 or §4 — walk one full protocol instance with actual sample values |
| Multi-lemma proof chain | §2 — apply one chosen instantiation of constants through Lemma 1 → 2 → 3 → main theorem |

### Worked illustrative example (a hypothetical world-model inference pipeline)

> **Running example (§4.1–§4.5).** observation `o` = 2×2 image `[[0.8, 0.2], [0.1, 0.9]]`, horizon `H = 3`, num candidates `K = 4`, goal latent `z_goal = (1.0, 0.5)`.
>
> **§4.1 Encode.** encoder(o) = z_0 = (0.7, 0.3) ∈ ℝ². _(Why these numbers? Linear encoder with W = [[0.5, 0.5, 0.0, 0.0], [0.0, 0.0, 0.1, 0.9]] applied to flatten(o).)_
>
> **§4.2 Sample candidates.** K=4 action sequences each of length 3. Show explicitly: a⁽¹⁾ = [+x, +x, +x], a⁽²⁾ = [+x, +y, −x], a⁽³⁾ = [+y, +y, +x], a⁽⁴⁾ = [−x, +y, +y].
>
> **§4.3 Imagine.** For each a⁽ᵏ⁾, predictor unrolls from z_0. Take a⁽¹⁾: predictor(z_0, +x) = (0.8, 0.3), predictor((0.8,0.3), +x) = (0.9, 0.3), predictor((0.9,0.3), +x) = (1.0, 0.3). Endpoint: z⁽¹⁾_end = (1.0, 0.3). (Same arithmetic for k=2,3,4 — show all 4 endpoints in a table.)
>
> **§4.4 Score.** cost_k = ‖z⁽ᵏ⁾_end − z_goal‖² for z_goal = (1.0, 0.5). cost_1 = (1.0−1.0)² + (0.3−0.5)² = **0.04**. Show cost_2, cost_3, cost_4 similarly.
>
> **§4.5 Select.** argmin_k cost_k = 1 (lowest). Emit first action of a⁽¹⁾ = +x.
>
> **Closing**: you just saw a full CEM rollout with concrete numbers. The paper actually uses K=512, H=15, z ∈ ℝ¹⁹², but every step is mechanically identical.

### Output discipline

- **≥1 running example mandatory** whenever the paper has any end-to-end pipeline (most do).
- **Same numbers throughout** — picking `z = (0.3, -0.5)` in stage 1 and silently switching to `z = (1.0, 1.0)` in stage 3 defeats the purpose.
- **Don't abandon mid-pipeline.** The most common failure: starting "Stage 1: encode → z_0 = (0.7, 0.3). Stage 2: predictor produces z_1. Stage 3: ..." (without the actual z_1). The reader is then forced to do their own arithmetic — exactly what the running example was supposed to save them from.
- **Stay Lego-scale.** The running example uses small `K`, small `H`, small dim. Paper-scale numbers belong in the closing summary line, not inside the running example.
- **Figure required.** A flow figure that shows the running-example value at every stage. Tag `[synthetic-demo]` (toy data) or `[reproduced]` (if you actually ran the released code with these inputs).

### Don't

- **Don't have multiple running examples that all do "one stage each".** That's just local toys with extra ceremony. The point is one example travels the whole pipeline.
- **Don't skip the closing "paper actually uses…" line.** Without it, the reader thinks the toy IS the paper's scale and gets confused by ablation numbers later.
- **Don't run the example at paper scale.** B=128, dim=192, K=512 doesn't fit on a page and the reader can't follow. Lego scale is the whole point.

### Interaction with the other moves

- This move **uses** #2 (Lego scale), #3 (toy demo), #10 (toolchain unfurling at each stage) — running example is the **integrator** that ties their outputs together end-to-end.
- For each stage, if the operation involves a new math tool, do the #10 unfurling for the tool **separately first**, then re-use it in the running example without re-explaining.
