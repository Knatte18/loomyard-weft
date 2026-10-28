# Lens sets and code review per profile as the review-setup audit recommends

`review-setup-tuning` · Task · enhancement, burler, stencil, config · running · Drift-Baseline

## Seen

`docs/research/review-setup-audit-2026-10-28/report.md` measured each lens and each review round over the landed loom runs.
The operator agreed with its lens and code-review recommendations (2026-10-28); this task builds them.

## Design

Lens sets (`internal/lenses`, the lens stencils under `contracts/stencils`):

- twill-discussion: merge `goal-scope` and `board-fit` into one board-and-scope lens that reports only what contradicts the board entry or another board entry; keep the other five lenses.
- review-plan: replace `way-forward` with a lens for repository invariants a card breaks without touching the line that breaks: test seams and tier tags, `doc.go` and PATTERN text a card falsifies, and card order. Keep `guard-reach` and `affected-set`.
- review-code: fold `code-pattern-fit` into `code-affected-set`, and narrow `code-tests` to missing coverage of changed behaviour, so it stops reporting test style.

Profiles: the Code review is a solo reviewer in the `light` and `docs` profiles; `solid` and `wide` keep the code lens fan.

## Out of scope

- Delta review of later rounds: every review round keeps reading the whole artifact, since two fresh reviewers of the same text find different defects and that redundancy is the point of a further round (operator, 2026-10-28).
- Round counts per profile stay as they are.
- New checks in `validate-plan` and `validate-discussion` are a task of their own.
