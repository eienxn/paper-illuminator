# Figure Contract (Reference)

> Companion to SKILL.md §Step 5 (fig=on). Covers **figure functional budget +
> visual style spec + Evidence Provenance Rule**.

---

## 1. Evidence Provenance Rule (the core quality rule)

**Every load-bearing claim and every figure must be tagged with one of**:

| Tag | Meaning | How to mark |
|---|---|---|
| `[paper-reported]` | Directly from paper's text / table / figure / appendix | "(paper Tab 2)" / "(paper Fig 5)" |
| `[repo-inspected]` | From the official code repo / config file | "(repo `config/train.yaml`)" |
| `[reproduced]` | You actually ran the released ckpt / experiment locally | "(local run, B=128)" |
| `[synthetic-demo]` | Toy data used to demonstrate a mechanism, **not the paper's real data** | "(toy demo, seeded)" / caption "illustrative reproduction" |
| `[interpretation]` | The paper didn't say this directly; this is your inference / framing | "My interpretation is ..." / "Can be understood as ..." |

### Hard rules (enforce when writing the deep note)

- **Forbid strong-claim words** like "demonstrates" / "proves" / "establishes" / "shows" — unless the source type is clearly one of `reproduced` or `paper-reported`.
- **Every number must be traceable to a tag**. e.g., "70% success rate" → `[paper-reported, Tab 1]`. e.g., "error grows 12× over 15 steps" → `[synthetic-demo, fig 06]`.
- **`[interpretation]` requires hedging first-person phrasing**: "My interpretation is ...", "Can be understood as ...", "**Likely** because ...". Never write "This shows X" in a way that makes the reader think the paper proved X.

### Figure caption rule

Every figure caption must lead with its provenance tag:

```markdown
![title](figures/NN_xxx.png)

**[synthetic-demo]** Panel-by-panel reading: ...

or

**[paper-reported, Fig 5 reproduced]** Panel-by-panel reading: ...

or

**[reproduced]** Loaded the released ckpt, ran on 1000 frames, computed PCA. Reading: ...
```

### ⚠️ `[synthetic-demo]` is a first-class tag — don't reduce toys because of this rule

This rule lets toys and real evidence **coexist without confusion**. It is not about reducing toys. **Toys / synthetic-demo figures are the tutor's core asset** — a 5-sample, 2-d, B=2 example the reader can simulate in their head is more pedagogically powerful than any paper-scale number.

**Right way**:
- Generate toy demos freely (counter-intuitive demos, Lego worked examples, one full algorithm iteration by hand, etc.)
- Add `[synthetic-demo]` to figure captions and next to numbers
- When useful, note "this is a toy; the paper actually uses ${scale}"
- Toys can sit **next to** real data in the same figure: left panel `[synthetic-demo]`, right panel `[paper-reported]`

**Wrong way** (what this rule fights):
- Passing off toy numbers as "measured" / "actual", making the reader think they're real evidence
- **Reducing toy demos out of fear that `[synthetic-demo]` looks "soft"** — this is the over-correction failure mode (and the bigger sin than over-using toys)
- Substituting a placeholder core operation (e.g., a simpler statistic standing in for the real one) without disclosure

**Core principle**: tag honestly, then toy freely.

---

## 2. Functional budget (replaces "≥N figures" hard count)

`depth=deep, fig=on` must satisfy the following functional minimums (counting matters less than coverage):

| Category | Min count | Preferred tag | Teaching move it serves |
|---|---|---|---|
| **Paper figure / table walkthroughs** | **≥4** | `[paper-reported]` | (engages the paper's own evidence) |
| **Worked example / numerical walkthrough** | **≥3** | `[synthetic-demo]` (toy values) | #2 Lego, #3 phenomenon demo |
| **Architecture / data flow / training flow** | **≥3** | `[paper-reported]` or `[interpretation]` | (orients the reader in the method) |
| **Results / ablation / failure mode** | **≥3** | `[paper-reported]` | (grounds claims in numbers) |
| **Real data / repo-inspected evidence** (when feasible) | **≥1** | `[reproduced]` or `[repo-inspected]` | #8 real-data evidence |
| **Motivation comparison figures** (3-panel: ground truth vs alternative vs paper's choice) | **≥2** | `[synthetic-demo]` or `[interpretation]` | #9 motivation walkthrough |
| **Toolchain unfurling toy figures** (the 2×2 toy at step D, for each core math tool) | **≥2** | `[synthetic-demo]` | #10 toolchain unfurling |
| **Running-example flow figure** (one figure where the running-example value is annotated at every stage) | **≥1** (if the paper has any end-to-end pipeline; 0 if it doesn't) | `[synthetic-demo]` or `[reproduced]` | #11 running example |
| **§1 background concept illustrations** (geometric intuition for a manifold / latent space / metric; concept-relationship map; analogy figure; neighboring-concept side-by-side; "where does this prerequisite live in the paper" anchor diagram) | **≥3** (at least one figure per non-trivial §1 prerequisite) | `[synthetic-demo]` or `[interpretation]` | §1 coverage; complements #2 worked example with non-numerical intuition figures |

**Minimum total**: 4 + 3 + 3 + 3 + 1 + 2 + 2 + 1 + 3 = **23** when the paper has a pipeline. **22** if no pipeline (drop the running-example row). **Cap**: 35. **Sweet spot**: 25–30.

**Pass criteria**:
- The sum above is the floor. Below this is a failure.
- ≥1 `[reproduced]` / `[repo-inspected]` figure is **strongly recommended**, not strict. If the paper has no public ckpt / requires large-scale hardware, you can skip it — but **state explicitly** in the deep note why you didn't include one.
- Running-example row: required when the paper has any end-to-end flow (training / inference / iterative algorithm / multi-step protocol / proof chain) — which is almost every paper. Only skip if the paper genuinely has no pipeline (e.g., a pure scaling-law observational study), and say so explicitly.

**Quality-over-count caveat**:
- The budget is a **minimum**, not a target. A 22-figure note where every figure answers a question the reader actually has beats a 32-figure note half of which are decorative.
- Don't pad by repeating the same information in different visual forms (3 bar charts of one ablation in different orderings, 2 versions of the same flow diagram with cosmetic differences). Each figure should be removable only at the cost of losing something a sentence can't replace.

**Failure modes to avoid**:
- 25 figures all `[synthetic-demo]`, never engaged with the paper's own figures → fails
- 23 figures all `[paper-reported]` redraws, no worked example / no motivation comparison / no toolchain toy / no background illustration → fails (not pedagogical)
- §1 background covered only in text with no figures — reader has no geometric / structural intuition to anchor the prerequisites → fails
- Any figure with no caption tag → fails
- Inflating the count with decorative redraws to hit the floor → fails (quality caveat above)
- Hitting cap=35 with 10 redundant ablation bar charts → fails (count is not the goal)

---

## 3. Paper figure/table audit step (mandatory, before Step 5.2)

**Before** generating any custom figure, do the audit:

```
In the deep note draft (or notes), list:
- Figure 1: ${one-line purpose}
- Figure 2: ${one-line purpose}
- ...
- Table 1: ${one-line purpose}
- ...

Then decide:
- Which paper figure/table is core evidence and deserves panel-by-panel reading (≥4)?
- Which is secondary and gets a one-line mention?
- Where does the paper leave something underexplained that needs a custom figure?

Then plan custom figures:
- Only to fill **teaching gaps** (background concepts / toy demos / worked examples).
- Do NOT reproduce evidence figures the paper already made well.
```

Write this audit into **Appendix A of the deep note** so the reader can trace your functional decisions.

---

## 4. Figure format selection (`--flow-style matplotlib` default for deep)

| Concept type | Format | Reason |
|---|---|---|
| Pipeline / data flow / system architecture | **matplotlib** (boxes + arrows + labels) → png | Richer visual; default for deep mode |
| Trivial 3–5 node inline flow | **mermaid** inline | Lighter; fine when not load-bearing |
| Tabular comparison | **markdown table** | Don't draw a figure for a table |
| Geometry / distribution / latent space | **matplotlib** → png | Mermaid can't draw scatter / contour |
| Algorithm with multiple stages | **matplotlib** multi-panel | Need "before / during / after" |
| Architecture with tensor/data shapes | **matplotlib** schematic with annotations (e.g., `(B, T, D)`) | Standard schematic format |
| Toy demo verifying a claim | **matplotlib** with seeded random data | Need actual numbers |
| Paper's original figure | embed extracted PDF/PNG from paper; add panel-by-panel reading in prose | Don't redraw |

When `--flow-style mermaid` is forced, use mermaid even for architecture. Default for deep is matplotlib.

---

## 5. Matplotlib flow style spec (when --flow-style matplotlib)

- **Rounded boxes**: `FancyBboxPatch` with `boxstyle="round,pad=0.05"`
- **Color coding by component role**:
  - input / data: light gray `#f0f0f0`
  - first-stage processor / encoder: light blue `#cce5ff`
  - secondary processor / predictor: light salmon `#ffd6cc`
  - third-stage / high-level processor: light orange `#ffe5b4`
  - control / action: light green `#d4f4d4`
  - loss / cost: pink `#fed9d9`
  - intermediate latent / sub-goal: light purple `#e6d4ff`
  - planner / algorithm: light yellow `#fff5cc`
  (The color names assume an ML-flavored pipeline; for non-ML domains, reuse the colors with role-equivalent meanings — e.g., for a theory paper, the "input" color marks assumptions, the "processor" color marks proof steps.)
- **Arrows**: `FancyArrowPatch` with `arrowstyle="->"`, mutation_scale ≈ 18
- **Shape annotations** on edges (e.g., `(B, D)` for ML tensors, or other domain-appropriate notation)
- **Multi-panel** when showing stages: "training view" vs "inference view", or "stage 1 / 2 / 3"
- **Title + legend**: always include
- **DPI ≥ 140**: otherwise screen captures look blurry
- **CJK font config** (only needed if the figure contains CJK characters, e.g. when `--lang chinese`):
  ```python
  import matplotlib
  matplotlib.rcParams['font.sans-serif'] = ['PingFang SC', 'Heiti SC', 'STHeiti', 'Arial Unicode MS', 'sans-serif']
  matplotlib.rcParams['axes.unicode_minus'] = False
  ```

---

## 6. Panel-by-panel reading (mandatory)

After every figure embed in the deep note, **immediately** follow with prose reading. No bare embeds.

Template:
```markdown
![title](figures/NN_xxx.png)

**[provenance tag]** Panel-by-panel reading:
- **Left panel**: ${what is shown / axes / color coding / key number}
- **Right panel**: ${...}
- **Key insight / the one number to remember**: ${...}
```

For single-panel figures, list at least 3 elements (axes / colors / annotations / key numbers).

---

## 7. Script saving

Every matplotlib figure's source script must be saved to `figures/NN_<name>.py`, named identically to the png (zero-padded). Reproducible by re-running the script.

Mermaid blocks live inline in the deep note; no separate script needed.
