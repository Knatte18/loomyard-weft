# Card 9 — batten-prune

**Edit:**
- `internal/battencli#`
- `internal/battenshed#`
- `internal/battenrecipe#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the batten packages' test files.

`internal/battencli` is one of the slowest in-scope packages; its `integration` files (`lifecycle_integration_test.go`, `recordleak_integration_test.go`, `records_integration_test.go`) build hubs and pairs, so they are the scenario-merge leads, and `PATTERN-batten-bookend` tests keep their shutdown-before-removal checks.
`TestRunCLI_GroupGuard_NoGitRepoNeeded` is a `failed` entry (Decision 8).
`internal/battenshed`'s candidates are fold leads over its step and refusal tests.

**ImpactSummary:** Test files of three batten packages only; no production change.

**Commit:** 9: Prune batten tests
