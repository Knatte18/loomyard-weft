# Card 16 — git-geometry-prune

**Edit:**
- `internal/fabriccli#`
- `internal/gitexec#`
- `internal/gitkit#`
- `internal/gitrepo#`
- `internal/gitrepo/internal/gitoracle#`
- `internal/githubclient#`
- `internal/hubforge#`
- `internal/hubgeom#`
- `internal/standalonegeom#`
- `internal/standalonestate#`
- `internal/lyxcwd#`
- `internal/pairteardown#`
- `internal/fslink#`
- `internal/fsx#`
- `internal/lock#`
- `internal/proc#`
- `internal/weftname#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the fabric CLI, git, hub-geometry, cwd and filesystem packages.
`internal/fabricengine` is core's and stays untouched (Decision 1); `internal/gitkit` and `internal/hubforge` are production test-support packages, not kits under `internal/testkit/`, so their own tests are this card's.

`internal/fabriccli` is among the slowest in-scope packages: its `integration` files each clone and wire a hub through `hubforge` (`PATTERN-hubforge-fixtures`), so they are the main scenario-merge leads, and its envelope-contract tests keep their `MutationRecord` key-set assertions (`PATTERN-mutation-record`).
`internal/gitrepo`'s parity tests pin the go-git versus `gitexec` boundary (`PATTERN-gitrepo-client-boundary`) and keep one row per operation.
`internal/pairteardown` keeps the reed-session-before-worktree order (`PATTERN-pair-teardown`).
`internal/gitkit`'s and `internal/hubforge`'s `bench_test.go` benchmarks are not tests the prune removes.
Every package here that spawns git keeps `gitkit.HermeticGitEnv()` (`PATTERN-test-isolation`).
`internal/githubclient` and `internal/proc` have `windows`-only test files, which this card leaves to their platform and vets only if it edits them.

**ImpactSummary:** Test files of the fabric, git, geometry and filesystem packages only; no production change.

**Commit:** 16: Prune fabric CLI, git and geometry tests
