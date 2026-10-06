# Card 17 — fabric-hub-prune

**Edit:**
- `internal/fabriccli#`
- `internal/hubforge#`
- `internal/pairteardown#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the fabric CLI, the hub fixture and the pair teardown.
`internal/fabricengine` is core's and stays untouched (Decision 1); `internal/hubforge` is a production test-support package, not a kit under `internal/testkit/`, so its own tests are this card's.

`internal/fabriccli` is among the slowest in-scope packages: its `integration` files each clone and wire a hub through `hubforge` (`PATTERN-hubforge-fixtures`), so they are the main scenario-merge leads, and its envelope-contract tests keep their `MutationRecord` key-set assertions (`PATTERN-mutation-record`).
`internal/pairteardown` keeps the reed-session-before-worktree order (`PATTERN-pair-teardown`).
`internal/hubforge`'s `bench_test.go` benchmark is not a test the prune removes.
Every package here that spawns git keeps `gitkit.HermeticGitEnv()` (`PATTERN-test-isolation`).

**ImpactSummary:** Test files of the fabric CLI, hub fixture and pair teardown packages only; no production change.

**Commit:** 17: Prune fabric CLI, hub fixture and pair teardown tests
