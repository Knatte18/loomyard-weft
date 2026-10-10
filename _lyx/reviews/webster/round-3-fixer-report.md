# Webster round 3 — fixer report

## Fixed

- **F1** (BLOCKING, scope) — `cmd/lyx/testdata/test-budget.yaml`: raised the `cmd/lyx` row from 73 to 74.
  The branch's `TestRootHook_NoVerboseFlagKeepsTheLevelSetAtInit` and main's `TestCLIForm_EveryCommand`, which the merge-in `d51290bb5` brought in, together put the package at 74 top-level tests.
  The finding's premise checks out against `.lyx/verify/verify-3.log`, where `cmd/lyx` is the only failing package.
  No test was added or removed, since the finding is a budget row and not a coverage gap.
  `lyx gate test ./cmd/lyx` passes.
  Commit `91bfad237`.
- **Gate: comment line breaks** — the comment gate flagged fixed-column breaks in comments the merge-in `d51290bb5` brought from main: `removeStatusStrands` in `internal/loomcli/bootstrap.go`, `TestApplyStatusStrandSurface` in `internal/loomcli/bootstrap_test.go` and `TestInterruptPolicies_MatchAssembledRows` in `internal/loomrecipe/interruptpolicy_meta_test.go`.
  Each is now broken at sentence and clause boundaries, with the wording unchanged.
  `lyx loom lint-comments --base 40cd76cf5` reports no finding.
  Commit `4c709c1ec`.

## Disputed

None.

## Deferred

None.
