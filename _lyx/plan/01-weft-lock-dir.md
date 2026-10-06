# Card 1 — weft-lock-dir

**Edit:**
- `internal/fabricengine#createWeftWorktree`
- `internal/fabricengine#createWeftWorktreeDroppingRuns`
- `internal/fabricengine#adoptWeftWorktree`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine/doc.go`
- `internal/hubgeom/doc.go`
- `internal/hubgeom/parent_test.go`
- `internal/hubgeom/reedgeom_integration_test.go`

**Uses:**
- `internal/fabricengine#ensureWeftLockDirAt`
- `internal/hubgeom#ResolveParent`
- `internal/fabricengine#KindDirCreated`

**Intent:**
Fixes `weft-lock-dir-drops-parent` (D1, D2, D3).

D1: every helper that runs a weft `git worktree add` calls `ensureWeftLockDirAt` on the new weft worktree right after the worktree exists and before it returns success.
That is `createWeftWorktree` (reconcile's dormant fork for a raw warp worktree), `createWeftWorktreeDroppingRuns` (Add's fork path, after the index write-out), `adoptWeftWorktree` (reconcile's `weft_recreated` path), and Add's own local-adopt branch right after its `containedWorktreeAdd`.
A failure returns an error from that helper (wrapped with the weft path), so Add rolls back through `rollbackAdd` and Reconcile reports the pair's `Error`, exactly as for a failed worktree add.

D2: in `Topology.Reconcile`'s per-pair loop, for a pair whose weft worktree already existed (`weftWorktreeExists`), before `repairPairWiring` runs: when `<weftPath>/.weft` is absent, call `ensureWeftLockDirAt`, record `KindDirCreated` at the lock directory's path in `rec`, and append a Detail note through `appendPrDetail` naming the restored weft lock directory.
It never sets or changes `pr.Action`: the action stays whatever `repairPairWiring` assigns (`already_healthy` included); a failure sets `pr.Error` with the cause.
No new `ReconcileAction` value and no envelope key.

D3: `ReadOriginFor` keeps answering `(Origin{}, false, nil)` when the weft worktree or the record file is absent.
When the record exists and the `.weft` directory does not, it returns an error naming the weft worktree path, saying its `.weft` lock directory is missing, and naming `lyx fabric reconcile` as the way forward.
It still creates nothing; update its doc comment accordingly.
`hubgeom.ResolveParent` already wraps and returns the error, and `Remove` already ignores it, so neither changes.
Bound: the only answer that changes is "record present, lock directory missing", which used to return a silent "no record"; no caller gains a write.

Docs: `internal/fabricengine/doc.go` gains a sentence that every weft worktree fabric creates gets its `.weft/` lock directory at birth, that reconcile restores a missing one, and that `ReadOriginFor` refuses a record without one; `internal/hubgeom/doc.go`'s `ResolveParent` sentence names the lock-directory error.

Tests:
- `internal/hubgeom/parent_test.go`: a row where `hubWithOrigin`'s record is present and the `.weft` directory is not; `ResolveParent` returns an error containing the lock-directory wording and `lyx fabric reconcile`.
- `internal/hubgeom/reedgeom_integration_test.go`, driving `fabriccli.RunCLIIn(..., []string{"reconcile"})` on a `hubforge` hub with a pair:
  (a) remove the pair's weft worktree (keeping the local weft branch), run `reconcile`, assert `.weft/` exists and `hubgeom.ResolveParent` yields the prime's orch name;
  (b) assert the freshly added pair's `.weft/` exists (the `add` fork path, which already creates it through the origin record's write), then delete only that `.weft/` directory, assert `ResolveParent` errors naming the missing lock directory and `lyx fabric reconcile`, run `reconcile`, assert `.weft/` exists, the parent resolves, and the pair's envelope `detail` names the restored lock directory while its `action` is `already_healthy`.
- No new test file covers `add`'s adopt-path `.weft/`: `add` writes the origin record on every path but an adopted weft that already carries one, so only card 4's record-carrying adopt row can fail without this card's `Topology.Add` edit, and it asserts `.weft/`; card 3's dormant-fork row asserts it for `createWeftWorktree`.

**ImpactSummary:** fabricengine weft-creation helpers, Add's adopt branch, Reconcile's per-pair loop and ReadOriginFor's gap answer; hubgeom callers inherit the new error, Remove ignores it.

**Commit:** 1: fabric: create .weft at weft birth, heal it in reconcile, refuse a record without it
