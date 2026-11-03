# Audit review findings for place and change, and skip the review bench

Adds `docs/research/concrete-review-findings-2026-11-03/`: an audit of Plan and Discussion review findings, in place of the planned eight-run review bench.

## What the change does

The board entry asked for a bench of the review stencil against a stencil with a rule that every finding names its place and the change.
`burler-review-format.md` has required that since `add9f93d`: every finding carries a `location` and a `fix`, "the exact change to make, and where".
The parent recorded the decision to audit first and bench only on real non-compliance.

The audit covers the 374 findings in the 44 Plan and Discussion review files written from 2026-10-31 to 2026-11-03.
All 374 have a file-naming `location` and a non-empty `fix`; 4 leave a choice or an investigation instead of one change.
The report recommends against adding the rule to the stencils, and notes that `fixers-on-sonnet`'s third arm has no input distinct from its second.

## Files

- `report.md`: method, result, recommendation, and the two rounds kept for `fixers-on-sonnet`.
- `data/findings.tsv` and `data/parse.awk`: every audited finding and the parser that produced it.
- `data/inputs.txt` and `data/fixers-input.tsv`: the two chosen rounds (weft and code commits) and their 24 single-change findings.

## Design decisions

- No review was run, so the report has no cost, cross-arm or per-run figures the entry listed; the parent's decision (`Audit before any review run`) replaces them.
- The audit judges form only: whether the named place is right or the change correct is not checked.
- Reviews before 2026-10-31 were not audited.
- The two kept rounds are a plan and a discussion round picked for having `BLOCKING` findings, since the entry named no artifacts.
