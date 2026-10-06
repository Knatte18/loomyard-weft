# Card 19 — git-fs-prune

**Edit:**
- `internal/gitexec#`
- `internal/gitkit#`
- `internal/gitrepo#`
- `internal/gitrepo/internal/gitoracle#`
- `internal/githubclient#`
- `internal/fslink#`
- `internal/fsx#`
- `internal/lock#`
- `internal/proc#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the git access, filesystem, lock and process packages.
`internal/gitkit` is a production test-support package, not a kit under `internal/testkit/`, so its own tests are this card's, and as a leaf package its tests keep its closed import set (`PATTERN-leaf-packages`).

`internal/gitrepo`'s parity tests pin the go-git versus `gitexec` boundary (`PATTERN-gitrepo-client-boundary`) and keep one row per operation.
`internal/gitkit`'s `bench_test.go` benchmark is not a test the prune removes.
Every package here that spawns git keeps `gitkit.HermeticGitEnv()` (`PATTERN-test-isolation`).
`internal/githubclient` and `internal/proc` have `windows`-only test files, which this card leaves to their platform and vets only if it edits them.

**ImpactSummary:** Test files of the git, filesystem, lock and process packages only; no production change.

**Commit:** 19: Prune git and filesystem tests
