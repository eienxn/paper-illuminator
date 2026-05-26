# {{TITLE}}

> **Authors:** {{AUTHORS}}
> **Venue / Date:** {{VENUE}} · {{DATE}}
> **Paper:** {{PAPER_URL}}
> **Repo:** {{REPO_URL}}  (or `not found`)
> **Reading depth:** {{DEPTH}} · **Figures:** {{FIG}}
> **Note generated:** {{TODAY}}
>
> See [glossary.md](glossary.md) for terms{{IF_QUESTIONS}}; see [questions.md](questions.md) for open follow-ups and gaps{{ENDIF}}.

---

## 1. One-line core
{{ONE_LINE_CORE}}

## 2. The question it answers
{{QUESTION}}

## 3. The claimed method
{{METHOD}}

**How it differs from mainstream methods:** {{VS_MAINSTREAM}}

## 4. Main claims
{{CLAIMS_LIST}}
<!-- Format: - [strong/medium/weak] <claim text> -->

## 5. Evidence for each claim + how much I believe it
{{CLAIM_EVIDENCE}}
<!-- Format: - **<short claim ref>** — evidence: <which table/fig/number>; judgment: <believe / partial / doubt / disbelieve> + one-line reason -->

## 6. Main baselines + datasets
{{BASELINES_AND_DATASETS}}

## 7. Compute requirements
{{COMPUTE}}
<!-- Param count, training GPU-hours, inference cost, memory. If unstated, mark as "unstated, estimated ~X" -->

## 8. Reproduction difficulty
{{REPRODUCIBILITY}}
<!-- repo URL, license, README quality (1-5), known gotchas, overall reproducibility (1-5) -->

## 9. Counter-intuitive points
{{COUNTERINTUITIVE}}
<!-- Format: - You might think X, but actually Y. Reason: ... -->

## 10. Limitations + failure modes
{{LIMITATIONS}}
<!-- Distinguish: what the paper itself admits / what you noticed it didn't admit -->

---

{{MAIN_BODY}}
<!-- This is the actual "explanation" body for middle / deep modes.
     Organize by the method's internal logic (NOT a 1:1 recap of the paper's sections).
     Every non-trivial concept hits ≥3 of {analogy, formula, worked example, figure, code}.
     Deep mode: each core section gets 1-2 "but what about…?" sub-questions.
     fig=on: interleave figures, each followed by a panel-by-panel walkthrough. -->

---

{{IF_DEEP}}
## suggested-survey-paragraph

> ⚠️ Draft for your review. Verify facts and wording before pasting into any external document.

{{SURVEY_DRAFT}}
{{ENDIF}}

{{IF_RELATED}}
## Related local notes
{{RELATED_NOTES}}
{{ENDIF}}
