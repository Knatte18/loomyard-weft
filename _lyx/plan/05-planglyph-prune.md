# Card 5 — planglyph-prune

**Edit:**
- `internal/planglyph#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/planglyph`'s test files.

The package has untagged tests and three `integration` files (`delta_integration_test.go`, `donecheck_integration_test.go`, `drift_integration_test.go`).
Tier-move leads (Decision 5) are the tests in those files that the report measured rather than listing as `spawns`; a test that needs a real git tree for its delta, drift or done-check stays tagged.
Tests in the integration files that each build the same seeded git worktree and differ only in the steps after it are scenario-merge leads (Shared Decisions, `prune-procedure`, step 4).

**ImpactSummary:** Test files of `internal/planglyph` only; no production change.

**Commit:** 5: Prune planglyph tests
