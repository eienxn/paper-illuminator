---
name: paper-illuminator
description: "Deeply read and explain a single academic paper like a patient tutor — mine unfamiliar concepts, build a glossary inline, generate many visual explanations, walk through every non-trivial step with analogies and simple worked examples. Outputs ONE consolidated markdown file that reads like a textbook chapter, with explicit evidence-provenance tags on every claim. Works across domains (theory, ML, wet-lab, systems, empirical). Use when user says \"deep read paper\", \"explain this paper\", \"help me really understand X paper\", \"paper tutor\", \"读这篇论文\", \"讲讲这篇 paper\", \"帮我吃透 X 论文\", or wants to genuinely understand a paper rather than just get a summary."
argument-hint: "<paper-url-or-path> [--depth skim|middle|deep] [--fig on|off] [--out DIR] [--context \"...\"] [--flow-style matplotlib|mermaid] [--lang english|chinese|...] [--iter N] [--with_ori] [--html] [--multi-file]"
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
> - `references/prose_style.md` — read-aloud-test rules for writing tutor-style prose (universal + Chinese / English specific)

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

**5. Visual verification + Symbol Mirror for paper figures.** When writing the panel-by-panel reading of an **extracted paper figure** (anything in `source-figures/`, especially `--with_ori` figures), you **MUST**:
   - **(a) Read the actual PNG with the Read tool BEFORE writing the caption.** Never describe a paper figure from memory or "what a generic figure of this type would look like". Visual hallucination (describing a layout that isn't there) is a hard failure mode — see the HWM Fig 3 incident: prose wrote "左侧 trajectory 灰条 + encoder $E$ 紫方块" for a figure that contained neither.
   - **(b) Mirror the figure's exact symbols.** If the figure labels nodes `z_{t_1}, z_{t_2}, z_{t_3}`, the prose says `z_{t_1}, z_{t_2}, z_{t_3}` — not the paper text's abstract `z_{t_k}, z_{t_{k+1}}, z_{t_{k+2}}`. You may add "(general form: $t_k, t_{k+1}, ...$)" as a *supplement*, but the primary description uses what's literally drawn. Mixing the figure's concrete indices with the paper text's abstract indices in the same caption is a hard failure mode.
   - **(c) Describe what's drawn, not what should be drawn.** If the figure omits a component (e.g. encoder $E$ silent / rollout loss arm absent), say so explicitly — don't fill it in from your mental model of the full system.

**6. Narrative spine — bridge between major sections.** Each major section (§1, §2, …, §6, §7) must include:
   - **(a) A 1–2 sentence "where we came from" recap** at section start when the next section's content depends on the previous. ("§1 gave you all the prerequisites — MPC, CEM, latent WMs, AR error. §2 puts them together as HWM's core structure.")
   - **(b) A 1-sentence "what's next" pointer** at section end. ("Now you've seen the structure; §3 walks through how to train it.")
   - **(c) Running example continuity (§4 inference and any pipeline section).** If §4.0 sets a running example with concrete numbers, every §4.x sub-section must explicitly re-reference those numbers ("recall from §4.1 that cost converged to 0.18 — here's the toolchain behind that"). Sub-sections that introduce a fresh unrelated example mid-section break cohesion.
   - **(d) Cross-link counter-intuitive points to their evidence.** §6 counter-intuitive items that depend on ablation evidence (e.g. latent dim sweet spot, transformer vs delta) must cite the §5.x section by number; conversely, §5 ablations that are also counter-intuitive findings should forward-link to the §6 entry.

   These bridges are what turns a list of sections into a coherent narrative. **Missing bridges = "feels piecemeal" failure mode.**

**7. Prose style — "read aloud" test.** The note's prose has to read like spoken explanation, not like notes / cheat sheets / table cells. Telegraphic comma-spliced fragments, dangling comparisons ("shorter than flat"), hidden bullet lists ("by (a)… and (b)…"), unverbalized symbol clusters ("H=2-5 steps"), and verbless claim fragments are all failure modes — they read like AI-compressed memos, not tutoring. See `references/prose_style.md` for the 8 universal rules + Chinese-specific / English-specific guidance + read-aloud test. Apply during writing, not as a post-pass — and never assume `writing-anti-ai` or `nature-polishing` will rescue bad prose mechanics (those skills catch AI tells, not rhythm).

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
- **DEFAULT_WITH_ORI** = `off` (when `on`, embed every original paper Figure/Table into the deep note at the most contextually appropriate section, each with panel-by-panel reading + connection to surrounding text)
- **DEFAULT_HTML** = `off` (when `on`, render the final deep note as a single browser-openable `.html` file via pandoc instead of `.md`; uses MathJax for LaTeX, mermaid CDN for diagrams, syntax-highlighting for code, sticky TOC sidebar)
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
| `--with_ori` | flag | absent | When set, every original Figure/Table in the paper must be (a) extracted into `source-figures/`, (b) embedded at the §X.Y of the deep note whose topic best matches it (not dumped into one gallery), (c) walked through panel-by-panel with provenance tag `[paper-reported, Fig N]` + a paragraph tying it to the surrounding §. Forces the agent to engage with every paper figure, which often surfaces details that would otherwise be skipped. See Step 5.4 for placement rules. |
| `--html` | flag | absent | When set, the final deliverable is rendered as a self-contained `.html` file (instead of `.md`) via pandoc. Pipeline: agent writes the markdown as usual, then `pandoc -s --toc --toc-depth=3 --mathjax --highlight-style=tango -c style.css note.md -o note.html` produces a browser-openable single file with sticky TOC sidebar, MathJax for LaTeX, syntax-highlighted code, and a mermaid CDN init for diagrams. Image paths stay relative — ship the `figures/` and `source-figures/` directories alongside the HTML. Requires pandoc installed (`command -v pandoc` to check). See Step 6.6. |
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

- **Text & metadata**:
  - **arXiv URL**: `/abs/` → `/e-print/` → curl `.tar.gz` → tar -xzf → locate `main.tex`/`paper.tex`. Also WebFetch abs page for title/authors/abstract.
  - **Local PDF**: prefer `pdf` skill; fallback `pdftotext`.
  - **Local .tex dir**: read directly.
  - **Repo**: search for repo URL in abstract / intro / footnote. If found, fetch README via `gh repo view` or WebFetch.

- **Figure extraction — fallback chain (try in order, log which one worked, stop at first success per figure)**:

  | # | Method | Best for | Quality | How |
  |---|---|---|---|---|
  | **1** | arXiv `.tar.gz` source figure files | arXiv papers | highest (lossless original) | locate `figures/`, `figs/`, `images/` or `*.{pdf,png,jpg,eps}` files inside the extracted tarball; copy into `source-figures/` |
  | **2** | Official repo's `figures/` / `assets/` / `paper/` dirs | papers with public GitHub repo | highest | `gh repo clone` or sparse-checkout the figure dir; many authors upload the paper-quality images |
  | **3** | arXiv HTML render at `ar5iv.labs.arxiv.org/html/<id>` | any arXiv paper | high | WebFetch the HTML, parse `<img>` tags, download the PNG/SVG URLs — figures are already extracted by ar5iv |
  | **4** | `pdffigures2` (Allen AI Java tool, region-aware) | any PDF | high | `pdffigures2 paper.pdf -o source-figures/ -i 0` — detects figure regions + captions intelligently. Needs Java + sbt; `command -v pdffigures2` to check |
  | **5** | `pdfimages -png paper.pdf source-figures/img` | any PDF | medium | extracts embedded bitmaps; misses vector/LaTeX-drawn figures; multi-panel figures get split into sub-files |
  | **6** | `pdftoppm -png -r 200 -f <page> -l <page>` (or `pdftocairo`) per-page render | vector/LaTeX-drawn figures #5 missed | medium (page-screenshot) | identify the page containing the missed figure (via `pdftotext` + text search for the caption), render that page; may need post-crop |
  | **7** | Publisher supplementary materials (Nature/IEEE/Springer/ACM) | journal papers with separate hi-res figure downloads | high | check the publisher landing page for "supplementary materials" / "source data" links |
  | **8** | Manual screenshot from PDF viewer | scanned/encrypted PDFs / everything else failed | varies | last-resort: ask the user to screenshot the figure(s) and place into `source-figures/` |

  **Implementation pattern** for each figure that needs extraction:
  ```
  for method in [arxiv_tarball, official_repo, ar5iv_html, pdffigures2,
                 pdfimages, pdftoppm_render, publisher_supplementary]:
      if method.available() and method.extract(figure_N) == success:
          log(f"figure {N} extracted via {method.name}")
          break
  else:
      log(f"figure {N} MANUAL EXTRACTION REQUIRED — note this in Appendix A")
  ```

  **Naming**: regardless of method, rename outputs to include figure number, e.g. `paper_fig3_attention.png` / `paper_tab2_results.png` — makes the file greppable when writing the deep note.

**Extraction policy by `--with_ori`**:
- `--with_ori off` (default): only extract figures you actually plan to embed (typically the ≥4 paper-figure walkthroughs in the figure budget). Skip the rest.
- `--with_ori on`: extract **every** Figure and Table from the paper using the fallback chain above. Build a complete inventory (used by Step 5.4). For any figure where all 7 automated methods fail (method 8 needs user), log it in Appendix A audit with the methods tried.

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

**Sub-step 5.4 — `--with_ori` placement rules** (only when `--with_ori` is set):

When `--with_ori` is set, **every** paper Figure/Table identified in the Sub-step 5.1 audit must be embedded into the deep note. The placement is **contextual**, not append-to-appendix:

1. **Assign each Figure/Table to its natural home** based on its topic, not its position in the paper:
   - Architecture / overview figure → §2.1 (Method overview)
   - Component-level diagram → §2.X (the matching component subsection)
   - Training pipeline / loss curve → §3 (Training details)
   - Inference flow / rollout figure → §4 (Inference / execution details)
   - Main results table / headline benchmark → §5 (Evaluation and findings)
   - Ablation figure / sensitivity sweep → §5 (the ablation subsection)
   - Failure-mode visualization → §7 (Pitfalls) or §6 (Counter-intuitive points), whichever fits
   - Conceptual / motivation figure → §1 (Background) or §2.0 (Motivation walkthrough)
   - Supplementary figure that doesn't fit anywhere obvious → embed in **Appendix F (Supplementary paper figures)** with a one-line note on what the paper used it for

2. **Each embedded paper figure must have**:
   - Provenance tag: `[paper-reported, Fig N]` or `[paper-reported, Tab N]`
   - Panel-by-panel reading (per `figure_contract.md` §6)
   - **A connecting paragraph** that ties the figure to the surrounding text — what does §X.Y's argument look like in this figure? what does the figure let the reader see that the prose can't say? This is the "the figure gets used, not just shown" rule.
   - The figure's `Notable detail / hidden assumption / counter-intuitive thing` if any (the *point* of `--with_ori` is that being forced to walk every figure surfaces details the agent would otherwise skip).

3. **De-duplication**: if two paper figures show essentially the same thing in different visual forms (e.g., bar chart and table of the same ablation), embed the more informative one and reference the other in one line.

4. **Cap and overflow**: hard cap **30 paper figures embedded in the body**. If the paper has >30 figures (long surveys / benchmark papers), embed the top 30 by load-bearing-ness and put the rest in Appendix F as a one-image-per-row gallery with one-line captions.

5. **Update Appendix A audit** to reflect that every paper figure was embedded (mark each row with `→ §X.Y` instead of `skip`).

6. **Interaction with figure budget** (`figure_contract.md` §2): `--with_ori` typically pushes the paper-figure-walkthrough count well above the ≥4 floor (which is fine). The custom-figure categories (architecture / motivation / toolchain toy / running example / §1 background) **still apply unchanged** — `--with_ori` adds paper figures *on top of* the custom figures, it does not replace them.

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
   - *"The figure shows `t_1, t_2, t_3` but the caption keeps saying `t_k, t_{k+1}, t_{k+2}` — which is right?"* OR *"The caption describes a 'left trajectory bar' but I don't see one in the figure"* → **Symbol Mirror / Visual Verification violation (rule #5)**. Re-open the PNG; rewrite the caption to use literal symbols + literal layout.
   - *"This section feels like a re-statement of the previous one — I'm not learning anything new"* → section overlap (e.g. §2.5 recapping §2.0). Merge, cross-link, or delete.
   - *"How does this section connect to the previous one? Why am I here now?"* → missing section bridge (rule #6). Add a 1-2 sentence "where we came from / what's next" hinge.
   - *"§4.0 set up a concrete running example but §4.3 introduces fresh numbers I don't recognize"* → running-example discontinuity (rule #6c). Re-thread §4.x sub-sections through the §4.0 numbers.
   - *"§6 lists a counter-intuitive point but doesn't tell me which experiment/figure backs it up"* OR *"§5 ablation surprised me but the §6 chapter doesn't list it"* → §5↔§6 cross-link missing (rule #6d).
   - *"This sentence has 3 commas and no verb — it reads like a spec-sheet row"* → rule #7 / prose_style 1.1 violation. Rewrite into 1-2 sentences each with subject + verb + object.
   - *"I had to mentally translate `H=2-5` into 'two to five' to parse the sentence"* → rule #7 / prose_style 1.2 violation. Verbalize the parameter; keep the symbol in parens as supplement.
   - *"Shorter than flat — flat's what?"* OR *"better than X — better at what?"* → dangling comparison, rule #7 / prose_style 1.3 violation. Complete the comparison.
   - *"(a) … + (b) … inside one sentence reads like a bullet list crammed in"* → rule #7 / prose_style 1.4 violation. Promote to real bullets or fully verbalize ("一是 …; 二是 …" / "first … second …").
   - *"Why is this sentence right after the previous one? What's the logical relation?"* → rule #7 / prose_style 1.6 violation. Add connective (因此/换言之/反过来 / Therefore/In other words/By contrast).
   - *"This paragraph reads like a table cell, not speech"* → rule #7 read-aloud test failed. Rewrite per `references/prose_style.md`.
   - *"中英 code-switch 每 3 个字一次, 我大脑卡了"* → rule #7 / prose_style 2.1 violation. Align switches with phrase boundaries; don't cut predicates between languages.

3. **Edit the existing sections in-place** to fix each friction point. **Do not append fixes as new sub-sections or addenda** — the reader on iteration N should see a smooth narrative, not a patch log. The patch log lives in Appendix E (next step), not in the body.

4. **Log the changes transparently into Appendix E (Self-iteration log).** For each fix: location | what the first-pass reader stumbled on | what was added/changed/moved. This lets the human reader trust that self-review happened, and lets future-you see where the typical friction points were.

5. **Early stop.** If a pass finds 0 friction points, stop — additional passes will not help. Always cap at 4 iterations regardless of `--iter` value.

#### Cap and sanity

- Hard cap: 4 iterations total (i.e., `--iter > 4` is treated as 4).
- A self-iteration pass that adds no fixes is allowed; record "0 friction points" in Appendix E and stop. Do not invent fixes to look busy.
- Self-iteration **only edits §1–§7 body and figures**. Appendices A–D should already be settled by the first pass; do not rewrite them in self-iteration unless a body fix forces a glossary or claim-ledger update.

### Step 6.6. (only if `--html` is set) Render markdown → HTML

After Step 6 (and Step 6.5 self-iteration if `--iter ≥ 2`) finishes producing the consolidated markdown file, run the pandoc render pipeline.

#### Preflight

- `command -v pandoc` to check pandoc is installed. If missing, stop with a clear message: "pandoc not found; install via `brew install pandoc` (macOS) / `apt-get install pandoc` (Linux) and re-run, or omit `--html` to keep the markdown deliverable."

#### Render command

```bash
pandoc \
  -s \
  --toc --toc-depth=3 \
  --mathjax \
  --highlight-style=tango \
  --metadata title="<title>" \
  -c style.css \
  -o "<title>_explained.html" \
  "<title>_explained.md"
```

Notes:
- `-s` produces standalone HTML (full `<html><head><body>`).
- `--toc --toc-depth=3` puts a table of contents at the top; depth 3 catches §X.Y.Z.
- `--mathjax` adds the MathJax CDN script so `$...$` and `$$...$$` render as math.
- `--highlight-style=tango` (or `pygments`, `kate`, `breezedark`) for syntax-highlighted code blocks.
- `-c style.css` links a small custom stylesheet (see below). If you want a single-file deliverable with no external deps, add `--embed-resources --standalone` and inline the CSS into the same file (larger but copy-pasteable).

#### Filename suffix follows `--lang`

- `--lang english` (default): `<title>_explained.html`
- `--lang chinese`: `<title>详解.html`
- Other languages: locale-appropriate suffix (e.g. `_erklart.html` for German).

#### Custom stylesheet `style.css`

Generate next to the HTML file. Aim for **comfortable long-form reading** — sticky TOC sidebar on wide screens, max-width body for line length, dark-mode friendly. A reasonable starting point:

```css
:root { --fg: #1a1a1a; --bg: #fefefe; --accent: #2c5aa0; --code-bg: #f4f4f4; --muted: #666; }
@media (prefers-color-scheme: dark) {
  :root { --fg: #e8e8e8; --bg: #1a1a1a; --accent: #7bb3ff; --code-bg: #2a2a2a; --muted: #999; }
}
body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans", sans-serif; line-height: 1.65; color: var(--fg); background: var(--bg); max-width: 820px; margin: 2rem auto; padding: 0 1.5rem; }
h1, h2, h3, h4 { line-height: 1.25; margin-top: 2rem; }
h1 { border-bottom: 2px solid var(--accent); padding-bottom: 0.3rem; }
h2 { border-bottom: 1px solid #ddd; padding-bottom: 0.2rem; }
img { max-width: 100%; height: auto; display: block; margin: 1rem auto; }
code { background: var(--code-bg); padding: 0.15em 0.4em; border-radius: 3px; font-size: 0.92em; }
pre code { background: none; padding: 0; }
pre { background: var(--code-bg); padding: 1rem; border-radius: 6px; overflow-x: auto; }
blockquote { border-left: 4px solid var(--accent); padding-left: 1rem; color: var(--muted); margin-left: 0; }
table { border-collapse: collapse; margin: 1rem 0; }
th, td { border: 1px solid #ddd; padding: 0.5rem 0.75rem; }
th { background: var(--code-bg); }
#TOC { position: sticky; top: 1rem; max-height: calc(100vh - 2rem); overflow-y: auto; font-size: 0.9em; }
@media (min-width: 1200px) {
  body { max-width: 1100px; display: grid; grid-template-columns: 250px 1fr; gap: 2rem; }
  #TOC { grid-column: 1; }
  body > *:not(#TOC) { grid-column: 2; }
}
.provenance-tag { font-family: monospace; font-size: 0.85em; color: var(--accent); }
```

#### Mermaid support

If the deep note has mermaid blocks (when `--flow-style mermaid`), inject the mermaid init script into the head via pandoc's `-H header.html`:

```html
<!-- header.html -->
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>
```

Then: `pandoc ... -H header.html ...`. The default markdown ```` ```mermaid ```` blocks become `<pre class="mermaid">` after pandoc — mermaid.js will render them client-side.

#### Image paths

Keep image references relative (`figures/01_xxx.png`, `source-figures/paper_fig3.png`). The HTML file expects the `figures/` and `source-figures/` directories to sit next to it. **Do not** rewrite paths to absolute — that breaks portability.

For maximum portability (single-file deliverable that survives email / Slack), use `--embed-resources --standalone` to base64-inline all images, CSS, and MathJax. Caveat: file size balloons (a 25-figure note → 5–20MB), and re-rendering becomes slower.

#### Output

When `--html` is set, the **primary deliverable** is the `.html` file. The intermediate `.md` is kept on disk next to it (for re-renders / editing); do not delete it.

#### Self-check after render

- Open the HTML in a browser (or `python3 -m http.server` if relative paths need a server) — verify (a) TOC renders, (b) math displays, (c) figures load, (d) mermaid blocks render if any, (e) code highlighting works.
- If math doesn't render: check that `$...$` wasn't accidentally written as `\(...\)` somewhere; pandoc with `--mathjax` expects dollar-sign delimiters.

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
21. **With `--with_ori`, skipping any paper Figure/Table.** Every figure and table from the paper must be embedded and walked through somewhere in the body (cap 30; overflow → Appendix F). "I didn't think Figure 7 was important enough" is not a valid reason — the whole point of `--with_ori` is forcing engagement with every visual the authors chose to include.
22. **With `--with_ori`, dumping all paper figures into one gallery.** Placement must be contextual — each figure goes to the §X.Y its topic matches (per Step 5.4). A gallery at the end of the note is the lazy fallback Appendix F, used only for figures that genuinely don't fit a body section.
23. **With `--with_ori`, embedding figures without a connecting paragraph.** The "the figure gets used, not just shown" rule: each embedded paper figure needs (a) panel-by-panel reading, (b) a paragraph tying it to the surrounding text's argument. Just dropping the image with the paper's original caption is insufficient.

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
