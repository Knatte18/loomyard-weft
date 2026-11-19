```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced what each decision meets in the code that already runs, and three findings come out of it.

**Paths traced, per decision**

- **D1, board branch fixed at `main`:**
  - `CloneHub` → `ensureBoardWorktree(weftPath, warpBranch, boardDir)` (`internal/fabricengine/clone.go:328`) names the board after the code default branch. **Breaks** under D3's "wires the hub as a two-repository clone does" (finding 2).
  - `remoteDefaultBranch` callers `cleanupRemoteWarp` (`cleanupremotewarp.go:87`) and `warpDefaultBranch` (`addadopt.go:60`) read the warp store only. Survives.
- **D1, records store configured with refspecs before its first fetch:**
  - `cloneRepo` (`clone.go:304`) is the existing weft clone, a default `git clone`. D3 says the store is initialised first. Survives, because D3 states the departure.
  - `probeWeftBinding` (`recordsprobe.go:60-120`) makes a `--single-branch` staged clone of origin `HEAD`. In a one-repository origin that is the code branch. Breaks only as the misstated check in finding 3.
- **D2, namespace on push, lease, delete and upstream:**
  - `DeleteRemoteBranch` (`gitrepo/push.go:308-325`): the short name cannot reach `refs/lym/`. D4 covers this. Survives.
  - `DeleteRemoteBranchLeased` (`push.go:345`) uses a `refs/heads/` lease, which D2's lease item covers. Survives.
  - `resolvePushDestination` (`pushdestination.go:43-68`) pushes to the URL, as D4's evidence says. Survives.
  - `cleanupRemoteWarp`, which runs `ls-remote --heads` and `--tags` on the weft (`cleanupremotewarp.go:93-101, 278, 294`), and `originreconcile`'s `ls-remote --heads` (`originreconcile.go:526`) are covered by D2's selection rule. Survive.
  - `checkSubpathBeforeReset` → `recordedAnchorOnWeft` (`resetsubpathcheck.go:26-50, 75-100`) stages the weft by URL with default refspecs and reads `refs/heads/&lt;prime&gt;-weft`. In a one-repository hub that misses the namespaced marker. **Breaks** (finding 1).
- **D2, remote `HEAD` read for the records store answers `main`:** `remoteDefaultBranch` is called only on warp stores. Survives.
- **D3, seed push at the board push's step:**
  - `CloneAndWire` (`fabriccli/clone.go:92-96`): `Bolt.PushRecorded` is the step D3 replaces. Survives.
  - `PushRecorded` defers and returns nil while the board write lock is held (`bolt.go:137`). A fresh clone holds no such lock. Survives.
  - The later `CommitAnchoredPaths` and the `hubreconcile.Ensure` walk (`fabriccli/clone.go:135, 161`) push the board again after the seed. Survives.
- **D3, `--single-repo` creates the records branch fresh:** `suffixWeftPrimaryBranch` (`clone.go:515-524`) adopts `origin/&lt;prime&gt;-weft` when the fetch brought one. D3 states it does not adopt. Survives as stated.
- **D3, a legacy two-repository weft with a board `.lym-anchor`:**
  - The `CloneHub` legacy adopt path (`clone.go:365-371`) is reached with no `--single-repo` and no `lym-bolt`. Survives.
  - `retireBoltAnchorMarker` runs on reconcile only (`anchormarker.go:186`). Survives.
- **D4, `ls-remote` before the plain delete:** `deleteRemoteBranch` (`destroy.go:1618-1635`) records a removal only when deleted is true. Survives.
- **D5, `lym.primeBranch` and the one-repository branches:** `reconcileMissingWeft`'s `oneRepositoryAt` early return (`reconcile.go:473`) goes with D5. Survives, since nothing then reads it.

**Findings**

1. **MEDIUM, design.** `_lym/discussion/decision-record.md:105-119` (D3).
   - A `--reset --subpath` clone of a one-repository hub passes the pre-reset subpath check.
   - The check stages the URL with default refspecs and looks for the anchor at `refs/heads/&lt;prime&gt;-weft`, then at code `HEAD`. Both miss `refs/lym/heads/&lt;prime&gt;-weft`, so a mismatched `--subpath` passes and `resetHub` removes the existing hub.
   - `CloneHub`'s adopt path then refuses the mismatch through `adoptRecordedAnchor`, so the operator has lost the hub the check exists to protect.
   - D3 does not mention `--reset`, and D2's rule against reading `refs/heads/` covers records-store paths but not this staged clone.
   - Evidence: `internal/fabricengine/resetsubpathcheck.go:35-50, 75-90`, `internal/fabricengine/clone.go:250-264, 361, 478-479`.
2. **MEDIUM, consistency.** `_lym/discussion/decision-record.md:42-43, 109-113` (D1 against D3).
   - D1 fixes the board branch as `main`. D3 says the one-URL clone "wires the hub as a two-repository clone does", and that `--single-repo` "seeds the records store as a fresh two-repository bind seeds its weft".
   - That existing bind path creates the board through `ensureBoardWorktree(weftPath, warpBranch, …)`, so the board takes the code default branch's name.
   - When the code default branch is `master`, the seed pushes the board to `refs/lym/heads/master`. D3's probe looks for `refs/lym/heads/main`, so a second machine's one-URL clone takes the two-repository path and treats the code repository as a weft.
   - D3 never says the clone departs from `warpBranch` when it checks out the board.
   - Evidence: `internal/fabricengine/clone.go:324-331`, `internal/fabricengine/boardweft.go:21-51`.
3. **LOW, consistency.** `_lym/discussion/decision-record.md:142-152` (D3).
   - D3 says the precondition check is one `git ls-remote` on origin. Yet three of its arms judge file contents on the default branch: `.lym-warp`, a root `.lym-anchor` and `_lym`.
   - `ls-remote` lists refs only. The existing reader of those files is `probeWeftBinding`'s staged `--single-branch` clone plus `git show`.
   - D3 misstates the mechanism and the network work of its refusal check.
   - Evidence: `internal/fabricengine/recordsprobe.go:71-74, 89-120`.

**Tool calls**

1. `add.go:440-470`, `ensureBoardWorktree` callers and `RecordsBranchName` callers: to find other paths that derive the board and records branch names, which the manifest lacked.
2. `cleanupremotewarp.go` `defaultBranch` uses and `resetsubpathcheck.go:55-110`: to see whether cleanup and the pre-reset check read the weft's `refs/heads/`, which the manifest lacked.
3. `reconcile.go:465-485` and `checkSubpathBeforeReset`'s head: to confirm the reset check stages by URL with default refspecs, and to see reconcile's one-repository branch. Neither was in the manifest.
