# paper-illuminator

A Claude Code skill that deeply reads and explains a single academic paper like a patient tutor — mines unfamiliar concepts, builds a glossary inline, generates many visual explanations, walks through every non-trivial step with analogies and **simple worked examples**. Outputs **one consolidated markdown file** that reads like a textbook chapter, with explicit **evidence-provenance tags** on every claim.

Works across domains: theory papers, ML method papers, wet-lab studies, systems papers, empirical studies, surveys, etc. The skill detects the paper's domain and applies the matching mining playbook — it does not force a one-size-fits-all ML checklist onto every paper.

## Install

User-level (all your projects):

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/eienxn/paper-illuminator ~/.claude/skills/paper-illuminator
```

Project-level (this project only):

```bash
mkdir -p .claude/skills
git clone https://github.com/eienxn/paper-illuminator .claude/skills/paper-illuminator
```

## Update

```bash
cd ~/.claude/skills/paper-illuminator && git pull
```

## Use

In Claude Code:

```
/paper-illuminator <paper-url-or-path> [--depth skim|middle|deep] [--fig on|off] [--out DIR] [--context "..."] [--flow-style matplotlib|mermaid] [--lang english|chinese|...]
```

Or in natural language: "deep read this paper", "explain this paper to me", "help me really understand X paper", "读这篇论文", "讲讲这篇 paper", "帮我吃透 X 论文".

### ⭐ Recommended: `--depth deep --fig on`

The skill is designed around this one setting. It's where it shines — patient, multi-modal, with **23–35 figures** (sweet spot 25–30) walked through panel-by-panel and every non-trivial step mined for hidden assumptions.

**Main example** — deep-read a paper directly from arXiv:

```
/paper-illuminator https://arxiv.org/abs/2603.19312 --depth deep --fig on
```

That's it. Use this as your default invocation.

**What it produces** (single consolidated file + figures dir):

```
paper-notes/
├── INDEX.md                          # auto-appended across reads
└── firstauthor2026_shortname/
    ├── <title>_explained.md          # THE deliverable — single textbook-chapter file
    │                                 #  (filename suffix follows --lang: e.g. 详解.md for chinese)
    ├── figures/                      # 23–35 figures with evidence-provenance tags (sweet spot 25–30)
    │   ├── 01_xxx.py + .png          # matplotlib flow diagrams, worked examples, results
    │   └── ...
    └── source-figures/               # optional, extracted from PDF
```

The single consolidated file is structured as:

- Preface + anchor concept + "where the anchor appears" table
- §1 Background (prerequisite concepts)
- §2 Method core (with §2.0 motivation walkthroughs for the load-bearing design choices)
- §3 Training details (if applicable)
- §4 Inference / execution details (with concrete worked examples)
- §5 Evaluation and findings
- §6 Counter-intuitive points (dedicated chapter)
- §7 Pitfalls / reproduction details
- §8 Assessment (only if `--context` was provided)
- Appendix A: paper figure / table audit
- Appendix B: Glossary
- Appendix C: Claim ledger (evidence provenance table)
- Appendix D: Summary card (10 fields)

### Why `deep + fig=on` is the default recommendation

| Setting | When it's useful | When it's wasted |
|---|---|---|
| `--depth deep --fig on` | You actually need to understand the paper (reproduce it, build on it, teach it) | If you just want a citation, this is overkill |
| `--depth middle --fig on` | Survey-level skim where you still want figures explained | Borderline — usually deep is worth the extra tokens |
| `--depth skim --fig off` | Triage: "is this paper worth reading at all?" | Almost never the final pass |

### Optional flags

- `--out <dir>` — write notes into a specific folder (e.g. `--out report` to collect all reads in a project's `report/`). Same `--out` across multiple papers means `INDEX.md` accumulates into a running table.
- `--context "..."` — tell the tutor what the reader already knows / why they're reading this. Only affects the final assessment section (§8); main body §1–§7 stays project-agnostic.
- `--flow-style matplotlib|mermaid` — pick the visual style for flow/architecture figures. `matplotlib` (default for deep) draws richer boxes-and-arrows in PNG; `mermaid` (default for middle) is text-based inline.
- `--lang english|chinese|...` — output language for all reader-facing prose (section headers, body text, figure captions). Default is `english`. When set to `chinese`, the consolidated file is named `<title>详解.md`; matplotlib scripts automatically configure CJK fonts. Provenance tags and code identifiers stay in their original form regardless of language.
- `--iter N` — number of writing passes (default `1`, capped at `4`). With `--iter 2+` the agent does a **first-pass-reader self-iteration loop** after the initial write: switches viewpoint, reads its own note as if for the first time, marks friction points (missing toolchain unfurling, dependency-chain skips, untagged claims, fait-accompli tool drops, etc.), and edits the body in-place. Each pass logs its findings into Appendix E (Self-iteration log). iter=2 catches most things; iter=3 catches subtle skips.
- `--with_ori` — force **every** original paper Figure/Table to be extracted into `source-figures/` and embedded into the deep note at the §X.Y whose topic best matches it (architecture figures → §2, training curves → §3, ablation tables → §5, etc.) — not dumped into one gallery. Each gets panel-by-panel reading plus a connecting paragraph tying it to the surrounding text. Cap 30 in body; overflow → Appendix F gallery. Extraction tries an 8-method fallback chain (arXiv tarball → official repo `figures/` → ar5iv HTML → pdffigures2 → pdfimages → pdftoppm page render → publisher supplementary → manual screenshot). Use this when you want maximum faithfulness to the paper's own visuals or when you suspect the agent would otherwise skip "less important" figures that actually hide details.
- `--html` — render the final deliverable as a `.html` file (instead of `.md`) via pandoc. Output is browser-openable with sticky TOC sidebar, MathJax for LaTeX, syntax highlighting for code, and mermaid CDN init for diagrams. Requires `pandoc` installed (`brew install pandoc` / `apt-get install pandoc`). Ship the `figures/` and `source-figures/` directories next to the HTML so relative image paths resolve. The `.md` source is kept on disk for editing / re-render. Default stylesheet `style.css` is at `templates/style.css` in the skill — auto-copied next to the HTML.
- `--multi-file` — legacy opt-in for the old split structure (`note.md` + `glossary.md` + `questions.md` + `bigpicture.md`). Almost never needed; the default single-file structure is recommended.
- **Local PDF path** instead of arXiv URL — works fine; figures from the PDF are extracted into `source-figures/` for reference.

## Layout

```
SKILL.md                              # main skill prompt (entry + workflow + hard rules)
references/
├── mining_playbooks.md               # Step A/B/C/D mining framework by paper domain
├── teaching_moves.md                 # 11 domain-agnostic teaching moves (incl. motivation walkthrough, toolchain unfurling, end-to-end running example)
└── figure_contract.md                # figure budget + visual spec + Evidence Provenance Rule
templates/
├── deep_note.md                      # single-file output template (default)
├── INDEX.md                          # auto-maintained running index template
├── note_legacy.md                    # legacy multi-file template (--multi-file only)
└── glossary_legacy.md                # legacy multi-file template (--multi-file only)
```

## License

MIT.
