# Card 8 — landingshed-prune

**Edit:**
- `internal/landingshed#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/landingshed`'s test files.

The package has a large candidate set with several `Removable: no` entries.
Refusal tests in a landing step pin their way-forward text (`PATTERN-refusal-way-forward`): each such text is a unique assertion under Decision 2 and survives as a row or in its own test.
The `integration` files (`finalize_integration_test.go`, `publish_integration_test.go`, `verifygate_integration_test.go`) hold the tier-move and scenario-merge leads: tests that each build one hub through `hubforge` and differ only in the landing steps after it are scenario candidates, and a hub fixture that stays is not hand-assembled (`PATTERN-hubforge-fixtures`).

**ImpactSummary:** Test files of `internal/landingshed` only; no production change.

**Commit:** 8: Prune landingshed tests
