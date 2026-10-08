# Card 13 — uncommitted-paths

**Create:**
- `plan:internal/websterengine#UncommittedPaths` -> `func UncommittedPaths(geom Geometry) ([]string, error)`

**Edit:**
- `internal/websterengine#AcceptBatchFabricReference`
- `internal/websterengine/doc.go`

**Uses:**
- `internal/websterengine#runOwnPath`
- `internal/websterengine#Git`

**Intent:** Give webster one reader of the worktree's uncommitted work that is not the run's own state, which the reset's envelope (cards 14, 15) and the recovery prompt (card 18) list for D4 and D5.
`UncommittedPaths`, in `auditledger.go` beside `runOwnPath`, returns the worktree's `DirtyPaths` (tracked changes and untracked files, worktree-relative as git status names them) minus every path `runOwnPath` calls the run's own state, sorted; a clean tree returns none, and a git read failure is returned.
It is exported because `webstercli` calls it for the reset's envelope, and it reads `geom`'s unexported `Git` in-package, so no other seam over `Geometry` is needed.
`AcceptBatchFabricReference`'s clean-tree evidence now calls it in place of its own `DirtyPaths` loop, so the filter exists once and accept-audit's dirt check reads exactly the list the reset reports.
Its behavior is unchanged: `TestAcceptBatchFabricReference_ToleratesTheRunsOwnState` and the "dirty worktree" row of `TestAcceptBatchFabricReference_Refusals` keep pinning the filter, and cards 14, 15 and 18 assert the list at their surfaces, so no test is added.
The package doc names the function among webster's worktree reads.

**ImpactSummary:** Moves accept-audit's dirt filter into one exported function, its one existing caller unchanged in behavior; cards 14, 15 and 18 are its new callers.
