---
name: paper-illuminator
description: "Deeply read and explain a single academic paper like a patient tutor — mine unfamiliar concepts, build a glossary inline, generate many visual explanations, walk through every non-trivial step with analogies and simple worked examples. Outputs ONE consolidated markdown file that reads like a textbook chapter, with explicit evidence-provenance tags on every claim. Works across domains (theory, ML, wet-lab, systems, empirical). Use when user says \"deep read paper\", \"explain this paper\", \"help me really understand X paper\", \"paper tutor\", \"读这篇论文\", \"讲讲这篇 paper\", \"帮我吃透 X 论文\", or wants to genuinely understand a paper rather than just get a summary."
argument-hint: "<paper-url-or-path> [--depth skim|middle|deep] [--fig on|off] [--out DIR] [--context \"...\"] [--flow-style matplotlib|mermaid] [--lang english|chinese|...] [--iter N] [--multi-file]"
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, WebFetch, WebSearch, Agent
---

# Paper Illuminator

Deeply read and **explain** an academic paper. **Input** is a paper (arXiv URL / PDF / .tex dir); **output** is **one consolidated `.md` file** that reads like a textbook chapter — patient, multi-modal, mined for hidden steps, with figures that get read panel-by-panel, every load-bearing claim tagged with its evidence source.

> **Style goal**: a smart-but-not-expert reader should be able to mentally simulate the method end-to-end after reading your output. Tutor, not summarizer.
>
> **Reference docs** (Read these before deep mode work):
> - `references/mining_playbooks.md` — Step A/B/C/D mining framework, loaded per paper domain
> - `references/teaching_moves.md` — 8 teaching moves (anchor / lego toy / phenomenon demo / counter-intuitive dialogue / multi-view / real-data evidence / etc.)
> - `references/figure_contract.md` — figure functional budget + visual style spec + **Evidence Provenance Rule** (core quality rule)

---

## Three non-negotiable rules

**1. Explain the paper in detail — not a polite summary.** A reader who never read the paper should be able to **mentally simulate** the method end-to-end after reading your output. Generic openings like *"The paper proposes X for Y, achieving SOTA on Z"* are a failure mode.

**2. Actively mine the points readers may not understand.** Do not wait for the reader to ask. Apply `references/mining_playbooks.md` Step A→D systematically. Pick the playbook rows that match the paper's domain — an ML systems paper hides depth in tensor shapes and hyperparam sources; a theory paper hides it in assumption tightness and counter-examples; a wet-lab paper hides it in controls and statistical assumptions. **Don't apply a fixed ML checklist to every paper.**

   **No-assumed-knowledge baseline (applies on iteration 1, not just iter ≥ 2).** Every construct — every symbol, every tool, every "obviously"-tagged step — must be unpacked as if the reader has never met it before. The skill's deliverable is the same regardless of `--iter`. The `--iter` self-iteration loop (Step 6.5) is *belt-and-suspenders* for catching genuinely subtle gaps the writer couldn't see — **not a safety net to lean on while writing the first pass less carefully**. If you wrote a passage thinking "they probably know this" or "I'll let iter=2 catch it", that passage already violated this rule.

   **End-to-end running examples (when the paper has a pipeline).** If the paper has any end-to-end flow — training pipeline, inference pipeline, iterative algorithm, multi-step protocol, multi-lemma proof — the deep note must include **at least one running example** that uses the **same concrete numbers** through every stage of the pipeline. Local toys (per component, per tool) are necessary but not sufficient; the reader also needs to see one set of numbers travel through the whole thing. See `teaching_moves.md` #11.

**3. Use apt examples + heavy visualization.** Every non-trivial concept hits ≥4 of {analogy, formula, **simple Lego worked example**, figure, code}. Concrete worked examples must be **Lego-level first** (`B=2, L=3, D=4` or domain-equivalent toy scale), then optionally scaled to paper-size. See `teaching_moves.md` #2.

   **Toy / synthetic-demo examples are a FIRST-CLASS asset, not a workaround.** They are how readers build intuition. Required minimum: ≥3 `[synthetic-demo]` figures + multiple inline lego worked examples for any deep note. The strictness in rule #4 below is about *labeling honestly*, not about reducing toys.

**4. Evidence provenance is non-negotiable.** Every load-bearing claim and every figure must be tagged with one of: `[paper-reported]` / `[repo-inspected]` / `[reproduced]` / `[synthetic-demo]` / `[interpretation]`. Never write "measured", "proves", "demonstrates", "core finding" without a clear source. See `figure_contract.md` §1 and the deep_note template appendix C (claim ledger).

   **Note**: `[synthetic-demo]` is a fully respected tag — it tells the reader "this is a toy I made to make the mechanism visible, not the paper's own data". A figure clearly tagged synthetic-demo is *better* than a figure that lies by silently dressing up toy data as real evidence. **Tag honestly, then toy freely.**

---

## Output philosophy: ONE file, like a textbook chapter

**Default**: all explanation lives in a **single consolidated** markdown file at `<out>/<paper-slug>/<title-slug>_explained.md` (or `<title-slug>详解.md` when `--lang chinese`), following the structure in `templates/deep_note.md`.

Includes inline:
- Table of contents (for deep)
- Preface + **anchor concept** + "where the anchor appears" table (teaching move #1)
- §1 Background (prerequisite concepts, each going analogy → simple lego → formula → figure)
- §2–§N Method body (in teaching order, NOT 1:1 with paper sections)
- §6 Counter-intuitive points chapter (each item in three-act format)
- §7 Pitfalls / reproduction details
- §8 Assessment (only when `--context` is provided)
- Appendix A: paper figure/table audit (transparency)
- Appendix B: Glossary
- Appendix C: **Claim ledger** (5 columns: claim | evidence | source-type | confidence | caveat)
- Appendix D: Summary card (10 fields)

**Do NOT split** unless `--multi-file` legacy mode. The legacy split templates (`note_legacy.md`, `glossary_legacy.md`) exist only for that opt-in.

---

## Constants

- **DEFAULT_DEPTH** = `middle`
- **DEFAULT_FIG** = `off`
- **DEFAULT_FLOW_STYLE** = `matplotlib` (when depth=deep) / `mermaid` (when depth=middle)
- **DEFAULT_OUTPUT_DIR** = `./paper-notes/`
- **DEFAULT_LANG** = `english`
- **DEFAULT_ITER** = `1` (number of writing passes; values ≥2 trigger first-pass-reader self-iteration loops; capped at 4)
- **CACHE_DIR** = `~/.cache/paper-illuminator/`
- **ARXIV_SRC_URL_PATTERN** = replace `/abs/` with `/e-print/` to get `.tar.gz`

---

## Inputs

Parse from `$ARGUMENTS`:

| Arg | Type | Default | Meaning |
|---|---|---|---|
| `<paper>` (positional) | string | required | arXiv URL **or** local PDF path **or** local `.tex` directory |
| `--depth` | `skim` / `middle` / `deep` | `middle` | Reading depth |
| `--fig` | `off` / `on` | `off` | Whether to generate figures |
| `--out` | path | `./paper-notes/` | Output dir |
| `--context` | string | (none) | Reader-specific context. ONLY affects the assessment section (§8). Main body §1–§7 stays project-agnostic. |
| `--flow-style` | `matplotlib` / `mermaid` | depth-dependent | Architecture / flow figure style. `matplotlib` is the deep-note style. |
| `--lang` | `english` / `chinese` / any human language | `english` | Output language for all reader-facing prose (section headers, body text, figure captions, appendix entries). Code identifiers, math symbols, and provenance tags stay in their original form. |
| `--iter` | int ≥1 (capped at 4) | `1` | Number of writing passes. `1` (default) writes the note once. `2`–`4` triggers post-write self-iteration loops: agent switches to a first-pass reader viewpoint, marks friction points in its own note, and edits the sections in-place. Each pass logs its findings into Appendix E (Self-iteration log). |
| `--multi-file` | flag | absent | Opt-in legacy split structure (note.md + glossary.md + questions.md). Almost never needed. |

If `<paper>` is omitted, ask the user. Everything else has a default.

---

## Workflow

### Step 1. Parse args, set up dirs

Decide `<paper-slug>` (e.g., `firstauthor2026_shortname`) and `<title-slug>` early. File suffix depends on `--lang`: `_explained.md` for English (default), `详解.md` for Chinese, or a locale-appropriate suffix for other languages. Create:

```
<out>/<paper-slug>/
├── <title-slug>_explained.md   # single consolidated file (suffix varies by --lang)
└── figures/                    # only if --fig on
<out>/INDEX.md                  # auto-maintained
```

### Step 2. Acquire paper source

- **arXiv URL**: `/abs/` → `/e-print/` → curl `.tar.gz` → tar -xzf → locate `main.tex`/`paper.tex`. Also WebFetch abs page for title/authors/abstract.
- **Local PDF**: prefer `pdf` skill; fallback `pdftotext`. Extract figures via `pdfimages -png` into `source-figures/`.
- **Local .tex dir**: read directly.
- **Repo**: search for repo URL in abstract / intro / footnote. If found, fetch README via `gh repo view` or WebFetch.

### Step 3. Plan the deep structure (for deep mode)

Draft a TOC matching `templates/deep_note.md`, **not** the paper's section order. Identify Step A domains (see `mining_playbooks.md`) and silently decide which playbooks load. **Do not render the Step A domain identification into reader output** — it is private planning.

### Step 4. Read at the chosen depth

#### `depth=skim`
Read abstract / intro / conclusion only. ~150-line single file with one-line core + main claims + compute cost + reproducibility + repo link. No figures, no mining playbook.

#### `depth=middle`
Read full paper. ~400–700 line file following the deep TOC compressed: §1 background 2–3 prerequisites max; §2 method ~1 paragraph per component; §3–5 each one section; §6+§7 bulleted. If `--fig on`: 3–6 figures.

#### `depth=deep` ← the skill's reason to exist
1. Read paper twice (structure pass + mining pass)
2. **Apply `mining_playbooks.md` Step A→D** (silently — the planning is private):
   - Step A: identify paper's domain(s)
   - Step B: walk universal mining checklist
   - Step C: load only the playbook rows whose domains were ticked
   - Step D: instantiate as code / figure / table
3. **Apply `teaching_moves.md` 11 moves** where they fit:
   - #1 anchor concept (mandatory)
   - #2 Lego before paper-scale (mandatory for every worked example)
   - #3 toy demo to SEE phenomenon (for any abstract claim)
   - #4 extreme-value thought experiment (for load-bearing parameters)
   - #5 counter-intuitive dialogue (three-act: naive → why wrong → actual + worked example)
   - #6 claim → deliver immediately (no forward refs)
   - #7 multi-view of core claim (multiple perspectives)
   - #8 real-data evidence when ckpt accessible
   - #9 motivation walkthrough — dedicated `## Why X?` chapter for each load-bearing design choice (≥2 per deep note, ≥4 of the 7 motivation moves each, sits *before* the method exposition). Mining-side raw material comes from `mining_playbooks.md` Step B "Design decision audit".
   - #10 toolchain unfurling — for every math tool or prerequisite construct (SVD, KL, Fisher info, attention weight matrix, etc.) walk the 4-step chain (A) materialize the object → (B) state what you want from it → (C) refresh the tool + justify "why this tool" → (D) compute on a 2×2 toy. No "we apply SVD to M" without A→D done first. Mining-side raw material comes from `mining_playbooks.md` Step B "Implicit prerequisites" (source / use / Lego toy).
   - #11 end-to-end running example — if the paper has any pipeline (training flow / inference flow / iterative algorithm / multi-step protocol / proof chain), declare **one** running example at the pipeline entry and carry the **same concrete numbers** through every stage with a stage-by-stage flow figure. Local toys (per component / per tool) are necessary but not sufficient — the reader also needs to see one set of numbers travel the whole pipeline. ≥1 running example mandatory whenever a pipeline exists.
4. **Write to `templates/deep_note.md` structure**. Fill all appendices A–D.
5. Apply `figure_contract.md` figure budget (Step 5).

### Step 5. (fig=on only) Paper figure audit, then generate

**Sub-step 5.1 — Audit paper figures/tables first** (per `figure_contract.md` §3):
- List every paper Figure/Table with one-line purpose
- Decide which to deep-read panel-by-panel (≥4)
- Decide which to skip
- Then plan custom figures only to **fill teaching gaps** (NOT to replace paper evidence)

Write the audit into Appendix A of the deep note (reader-facing transparency about which paper figures you deep-read vs skipped).

**Sub-step 5.2 — Generate figures** following `figure_contract.md` §2 functional budget:
- Paper figure/table walkthrough: **≥4** with `[paper-reported]` tag
- Worked example: **≥3** with `[synthetic-demo]` tag (Lego-level toy values)
- Architecture/data flow/training flow: **≥3** in matplotlib flow style (see `figure_contract.md` §5)
- Results/ablation/failure mode: **≥3** with `[paper-reported]` tag
- Real data / repo-inspected evidence: **≥1** if feasible
- Motivation comparison figure (3-panel, per `teaching_moves.md` #9): **≥2** with `[synthetic-demo]` or `[interpretation]` tag
- Toolchain unfurling toy (the 2×2 step-D toy of `teaching_moves.md` #10, for each core math tool): **≥2** with `[synthetic-demo]` tag
- Running-example flow figure (per `teaching_moves.md` #11, one figure with the running value annotated at each stage): **≥1** when the paper has any pipeline (which is almost every paper)
- **§1 background concept illustrations** (geometric intuition / concept-relationship map / analogy figure / neighboring-concept side-by-side): **≥3** — at least one figure per non-trivial §1 prerequisite. These are the non-numerical intuition figures that anchor each background concept *before* worked examples; complements but does not replace the worked-example category above.

Total minimum: **23 figures** when the paper has a pipeline (22 without). **Cap**: 35. Sweet spot: **25–30**.

**Sub-step 5.3 — Every figure gets a provenance tag + panel-by-panel prose walkthrough** (see `figure_contract.md` §6).

### Step 6. Write the consolidated main file

Follow `templates/deep_note.md`. Keep the final self-check from that template — it's the writing-time gate.

**Language**: all reader-facing prose (section headers, body text, figure captions, appendix entries) must be in the language specified by `--lang`. Default is English. Code identifiers, math symbols, and provenance tags (`[paper-reported]` etc.) stay in their original form regardless of `--lang`.

**Do NOT render agent-private meta-content** (Step A domain identification, "which playbooks I loaded", self-check checklist) into the reader output. Those live in HTML comments in the template — they are QA, not content.

### Step 6.5. (only if `--iter` ≥ 2) Self-iteration loop

After the first pass of the consolidated file is written, run `(--iter - 1)` additional self-iteration passes. Each pass is **not** a re-write — it is a **viewpoint switch + targeted edit-in-place**.

#### Why this exists

Writing the note and reading the note are different cognitive modes. After writing, you know *why* every sentence is there, so you can't see what the first-pass reader will trip over. The viewpoint switch surfaces friction points that were invisible to the writer. Iteration 2 catches most; iteration 3 catches the subtle skips; beyond 4, additional passes mostly add noise.

#### One self-iteration pass

1. **Switch viewpoint.** Pretend you have never read this paper. You are a smart-but-not-expert reader opening the note for the first time. Read it from the top, in order, without skipping.

2. **Mark friction points** — places where you, as the first-pass reader, would have stopped or stalled. Common patterns:
   - *"Wait, what is M / X / this symbol?"* → step A of toolchain unfurling (#10) missing — the object was never materialized before being used.
   - *"Wait, why is the paper doing this?"* → motivation walkthrough (#9) missing, OR dependency chain skipped (jumped from method to consequence without the bridge).
   - *"I have to take this on faith — there's no evidence tag."* → Evidence Provenance Rule (rule #4) violated.
   - *"The derivation jumped from line N to line N+1 — I can't reconstruct the intermediate step."* → skipped algebra; expand.
   - *"I don't have a concrete mental picture of what this looks like."* → missing Lego toy (#2) or step D of toolchain unfurling (#10).
   - *"This figure has no provenance tag / no panel-by-panel reading."* → violates `figure_contract.md` §1 and §6.
   - *"The text says 'demonstrates' / 'shows' / 'proves' but there's no source nearby."* → rule #4 violation; add hedge or source.
   - *"Acronym appeared without first-mention definition."* → not in Appendix B glossary; fix.

3. **Edit the existing sections in-place** to fix each friction point. **Do not append fixes as new sub-sections or addenda** — the reader on iteration N should see a smooth narrative, not a patch log. The patch log lives in Appendix E (next step), not in the body.

4. **Log the changes transparently into Appendix E (Self-iteration log).** For each fix: location | what the first-pass reader stumbled on | what was added/changed/moved. This lets the human reader trust that self-review happened, and lets future-you see where the typical friction points were.

5. **Early stop.** If a pass finds 0 friction points, stop — additional passes will not help. Always cap at 4 iterations regardless of `--iter` value.

#### Cap and sanity

- Hard cap: 4 iterations total (i.e., `--iter > 4` is treated as 4).
- A self-iteration pass that adds no fixes is allowed; record "0 friction points" in Appendix E and stop. Do not invent fixes to look busy.
- Self-iteration **only edits §1–§7 body and figures**. Appendices A–D should already be settled by the first pass; do not rewrite them in self-iteration unless a body fix forces a glossary or claim-ledger update.

### Step 7. Update INDEX.md

```
| <YYYY-MM-DD> | [<title>](<paper-slug>/<title-slug>_explained.md) | <first-author> <year> | <one-sentence-core> | <depth> | <fig> |
```

Sort by date desc.

### Step 8. (optional) Related notes

If sibling paper-slug notes exist, grep for shared keywords and add cross-links at bottom of new note. If nothing relevant, write nothing.

---

## Output directory structure (deep + fig=on)

```
<out>/
├── INDEX.md
└── <paper-slug>/
    ├── <title-slug>_explained.md   # THE deliverable (suffix varies by --lang)
    ├── figures/                    # only if --fig on
    │   ├── 01_<name>.py + .png
    │   └── ... (≥23 figures for deep when paper has a pipeline; sweet spot 25–30)
    └── source-figures/             # optional, from PDF extraction
```

---

## Failure modes (DON'T do these)

1. **"The paper proposes X for Y" opening sentence** — replace with concrete one-line core followed by anchor concept.
2. **Bare figure embed with no provenance tag + no panel-by-panel reading** — fix both.
3. **Acronym used in main file with no first-mention definition** — add to glossary appendix B, cross-reference.
4. **"measured" / "proves" / "demonstrates" / "core finding" without provenance tag** — violation of evidence rule (rule #4). Fix immediately.
5. **Splitting into multiple files by default** — single file unless `--multi-file`.
6. **Echoing the paper's section order 1:1** — synthesize into teaching order, not citation order.
7. **Forward references** like "see §5" — claim → deliver immediately (teaching move #6).
8. **Going straight to paper-scale numbers in worked examples** — Lego first (B=2, L=3, or domain-appropriate toy scale) then scale. See `teaching_moves.md` #2.
9. **Bleeding `--context` content into §1–§7** — context firewall: reader-specific framing only in §8. Final self-check must verify no lab / survey / "should we include this" / follow-up plan language in main body.
10. **<23 figures for `depth=deep, fig=on`** (or <22 if the paper has no pipeline) — violates `figure_contract.md` §2 functional budget. Sweet spot is 25–30.
11. **Generating 20+ figures all `[synthetic-demo]`** — paper figure/table walkthroughs must be ≥4 with `[paper-reported]`. Failure to engage the paper's own evidence.
12. **§1 background covered text-only with no figures** — the reader has no geometric / structural intuition to anchor each prerequisite. Every non-trivial §1 prerequisite needs at least one figure (geometric intuition / concept-relationship map / analogy figure / neighboring-concept side-by-side).
12. **Toy code that silently substitutes the actual core operation** — toy can scale down (B=2 instead of B=128), must NOT silently replace the actual algorithm's core operation with a placeholder. If you simplify, explicitly mark it "(simplified for illustration — actual paper uses X)". **Making toys is encouraged. The failure mode is misrepresenting them, not making them.** If the toy IS the algorithm at smaller scale, that's perfect. If the toy uses a placeholder operation (e.g. `ks_test` standing in for the actual Cramér distance), say so.
13. **Skipping anchor concept in the preface** — teaching move #1 is mandatory for deep.
14. **Skipping paper figure/table audit before custom figures** — Step 5.1 is mandatory.
15. **Applying wrong mining playbook to paper** — apply only domains identified in Step A. Don't pad with patterns that don't fit.
16. **Rendering Step A domain identification or self-check meta into the reader output** — those are private planning. Reader-facing transparency belongs in Appendix A (paper figure/table audit), not in a "loaded playbooks" or "domain detection" section.
17. **Writing reader prose in a language other than `--lang`** — if `--lang chinese`, all section headers and body prose must be in Chinese; if `--lang` is omitted, default to English. Provenance tags and code identifiers stay in their original form.
18. **Using the `--iter` self-iteration loop as an excuse to write a careless first pass.** The no-assumed-knowledge baseline applies on iteration 1 — iter ≥ 2 is for catching subtle gaps the writer couldn't see, not for filling in things the writer knew to explain but skipped. A note where iter=1 violates the baseline is a failure even if iter=2 fixes it.
19. **Pipeline exposition without a running example.** If the paper has an end-to-end flow (training, inference, iterative algorithm, protocol, proof chain), and the deep note presents it stage-by-stage with no single concrete dataset traveling through all stages — the reader has to mentally invent their own numbers at each transition. Use `teaching_moves.md` #11: one running example, same numbers carried end-to-end.
20. **Switching numbers mid-running-example.** Stage 1 uses `z = (0.3, -0.5)`, then Stage 3 randomly switches to `z = (1.0, 1.0)` without saying why — the reader can't connect the stages. Pick one set of toy numbers and carry them to the end.

---

## Good vs Bad: a worked example

Suppose the paper introduces a regularizer called `SIGReg` that constrains encoder outputs to approximate an isotropic Gaussian, preventing representation collapse.

**Bad** (summary style, no provenance, no toy):
> The paper introduces SIGReg, a regularizer that encourages the encoder output to follow an isotropic Gaussian distribution. This prevents representation collapse and outperforms prior methods.

**Good** (tutor style — inside a larger §1 background that already explained "what is collapse"):

> ### 4.3 SIGReg: how does it enforce "looks like a Gaussian"?
>
> SIGReg is the only mechanism the paper uses to prevent latent collapse — it samples K random directions across the batch, projects each latent onto that direction, runs a "does this 1-D distribution look like N(0,1)?" statistical test, and pushes the deviation back as a loss. **[paper-reported, §4.3]**
>
> #### Intuition
> Imagine all latents collapsed near the origin: along every direction, the projections are ~0, so the empirical distribution is δ(0) — maximally far from N(0,1), so the loss explodes. To reduce the loss the model has to spread latents out into something ball-shaped.
>
> #### Lego-level worked example (B=4, latent dim=2)
> ```
> Collapsed batch: z = [(0.1, 0.0), (0.0, 0.1), (-0.1, 0.0), (0.0, -0.1)]
> Direction d = (1, 0). Projections = [0.1, 0.0, -0.1, 0.0]. Empirical CDF
> is concentrated near ±0.1.
> True N(0,1) CDF spreads across [-3, 3]. Cramér distance = ∫(F̂ − Φ)² dy ≈ 0.9.
>
> Spread-out batch: z = [(1.2, 0.3), (-0.8, 1.1), (0.5, -1.4), (-0.9, 0.2)]
> Projections = [1.2, -0.8, 0.5, -0.9]. Empirical CDF ≈ N(0,1).
> Cramér distance ≈ 0.1.  Loss drops 9×.
> ```
>
> #### Formula (simplified pseudocode)
> ```python
> for k in range(K):
>     d = random_direction()                              # unit vector
>     projections = latent_batch @ d                      # (B,) 1-D vector
>     loss += cramer_distance(projections, Normal(0, 1))  # ∫(F̂ - Φ)² dy
> # The paper uses Cramér distance (L² distance between CDFs),
> # NOT a KS test (sup distance). K (# of projections) and B (batch size)
> # are both critical hyperparameters.
> ```
>
> #### ⚠️ Pitfall (paper-vs-repo gap)
> K (number of projection directions) cannot be too small. Paper Appendix D defaults to K=16; at K=4 the variance is high enough that the test misjudges often; at K=64 the test is stable but training is 2× slower. This hyperparameter is not emphasized in the paper text but is hard-coded in `config/train.yaml` in the repo. **[repo-inspected]**

**Key difference**: the tutor version delivers (a) a provenance tag, (b) intuition, (c) a Lego worked example with concrete numbers, (d) formula with an explicit simplification flag, (e) a repo-inspected gotcha. The summary version delivers none of these.

(The example above is illustrative — apply the same pattern to whatever the paper's actual core mechanism is, whether that's a regularizer, a proof step, a training recipe, a dataset construction protocol, etc.)

---

## When NOT to use this skill

- Literature **search** → use `arxiv` / `semantic-scholar` / `research-lit`
- **Peer review** → use `academic-paper-reviewer`
- **Reproduction** → this helps understanding, not running
- **Novelty check** → use `novelty-check`
- Batch processing of many papers → single-paper-deep; 50 in a loop = shallow output
