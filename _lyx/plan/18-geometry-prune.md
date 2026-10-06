# Card 18 — geometry-prune

**Edit:**
- `internal/hubgeom#`
- `internal/standalonegeom#`
- `internal/standalonestate#`
- `internal/lyxcwd#`
- `internal/weftname#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the hub, standalone and cwd geometry packages and `internal/weftname`.

`internal/standalonestate` is a leaf package: its tests keep its closed import set (`PATTERN-leaf-packages`).
Every package here that spawns git keeps `gitkit.HermeticGitEnv()` (`PATTERN-test-isolation`).

**ImpactSummary:** Test files of the hub, standalone and cwd geometry packages and `internal/weftname` only; no production change.

**Commit:** 18: Prune geometry tests
