# Glossary — {{TITLE}}

> All acronyms, proper nouns, and notation used in this paper. Scan once before reading [note.md](note.md).
>
> Any term used in note.md without a definition here is a bug — please come back and add it.

## Acronyms

| Acronym | Full name | One-line gloss |
|---|---|---|
| {{ACRONYM}} | {{FULL_NAME}} | {{ONE_LINE_GLOSS}} |

<!-- Example:
| JEPA | Joint-Embedding Predictive Architecture | Predicts the next latent in latent space rather than decoding back to pixels. |
| SIGReg | Sketched Isotropic Gaussian Regularization | A collapse-prevention regularizer that pushes latent projections along random directions toward N(0,1). |
-->

## Proper nouns

| Term | Definition | Role in this paper |
|---|---|---|
| {{TERM}} | {{DEFINITION}} | {{ROLE_IN_PAPER}} |

## Notation

| Symbol | Meaning | Shape / type |
|---|---|---|
| {{SYMBOL}} | {{MEANING}} | {{SHAPE}} |

<!-- Example:
| $z_t$ | Latent representation at time t | $\mathbb{R}^{192}$ |
| $g$ | Predictor network | $\mathbb{R}^{3 \times 192} \times \mathbb{R}^{192} \to \mathbb{R}^{192}$ |
-->

## Neighboring concepts (commonly confused)

| Don't confuse | A | B | Key difference |
|---|---|---|---|
| {{A_VS_B}} | ... | ... | ... |

<!-- Example:
| EMA encoder vs shared encoder | BYOL's target encoder is updated by EMA from the online encoder | A shared encoder has both branches use the same weights and both receive gradients | Determines whether removing the EMA loop causes representation collapse |
-->
