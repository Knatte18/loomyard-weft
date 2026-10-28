---
format: 5
approved: false
---

# Plan: review-setup-tuning

Build the accepted lens and code-review recommendations of `docs/research/review-setup-audit-2026-10-28/report.md`:
open loom's `code_reviewer_fan` to the local layer, run the code review solo in the `light` and `docs` profiles and `speed`'s loom block, merge `goal-scope` and `board-fit` into `board-scope`, put `repo-invariants` in `review-plan` in place of `way-forward`, fold `code-pattern-fit` into `code-affected-set` and narrow `code-tests`.

## Card Index

1 — code-reviewer-fan-local — reclassify `code_reviewer_fan` to the local layer, with its tests and the docs that call it closed
2 — solo-code-review-profiles — set `code_reviewer_fan: ""` in the `light`, `speed` loom and `docs` blocks, with runprofile's doc
3 — board-scope-lens — replace `goal-scope` and `board-fit` with the procedure lens `board-scope`
4 — repo-invariants-lens — add `repo-invariants` to `review-plan` in place of `way-forward`, which gains the emphasis line
5 — code-review-lenses — fold `code-pattern-fit` into `code-affected-set` and narrow `code-tests`

## Shared Decisions

### Decision: reclassify before the profiles

Card 1 runs before card 2, because configreg checks each profile block through `configengine.ValidateLocal`, which refuses a guard key: `TestProfilesCheckLoad`'s template row fails at the batch and `webster` gates without card 1, since `internal/configreg` is outside card 2's packages.

### Decision: template checks as helpers

The criteria that the emphasis-line exemption and the writer-covers-reviewer check still fail for a lens they must not admit cannot be reached through the template alone, which holds no such lens.
Card 3 moves the emphasis check into the test helper `emphasisLineGaps` and card 4 moves the writer check into `writerFanGaps`; `TestConfigTemplate_DecodesAndIsSelfConsistent` calls both over the template, and `TestTemplateChecks_FlagBrokenTemplates` drives them over hand-built maps that must fail.

### Decision: one card per lens decision

D1, D2 and D3 with D4 each edit `internal/lenses/template.yaml` and `TestConfigTemplate_DecodesAndIsSelfConsistent` in turn, each card setting the lens count and fan orders its own change leaves, so every card passes its own gate.
