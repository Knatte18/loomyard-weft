```yaml
lens: code-correctness
outcome: reported
```

Task restated: lens code-correctness over the hub-pull diff, one concrete case per changed function or branch.

**Per-function list (case run → verdict)**

- `PullHub`, normal input (all sides present) → correct: order is Bolt, prime, beams; the deferred snapshot records mutations on every return path.
- `PullHub`, `localBeamLocations` fails (unreadable worktree list or origin record) → returns early with the Bolt and prime sides kept. Correct for a list failure; see finding 2 for the per-slug case.
- `pullBoardSide`, `SkipGit` → `Bolt.Pull` returns empty, so the side reads `up_to_date`. Correct.
- `pullBoardSide`, no upstream → `up_to_date`. Correct.
- `pullBoardSide`, `fetch_failed` skip → `stopped` with the way-forward detail. Correct.
- `pullBoardSide`, lock error → `stopped` with the raw error text only, no way forward. Minor, not a defect.
- `Bolt.Pull`, board write lock then `pullFirst` taking the push lock → order matches the decision. Correct.
- `pullPairSide`, beam with no upstream (`upstreamSHAAt` on `hub/&lt;name&gt;`) → `up_to_date`, no lock taken. Correct; `WorktreePath()` is `hub/&lt;name&gt;`, the same checkout `acquireBranchLock` reads.
- `pullPairSide`, detached beam HEAD → reported `up_to_date` ("no upstream"). Acceptable.
- `pullPairSide`, `Open` error or lock error (including Ctrl-C cancelling the context) → `stopped`, and the later sides still run. Correct.
- `pullPairSide`, `Fabric.Pull` error → mutations extended, then `stopped`. Correct.
- `pullPairSide`, weft not reached → `stopped` with a way forward. Correct, except for `SkipGit` (finding 1).
- `movedOrCurrent` → correct, assuming `Mutated()` holds only entries from a real move.
- `localBeamLocations`, prime in the worktree list → skipped unless its origin record carries the beam mark. Correct.
- `runHubPull`, a stop → error envelope carrying all sides and the stopped list. Correct.
- `runHubPull`, `PullHub` returns an error → the partial sides are still reported. Correct.
- `acquireBranchLock` with a verb parameter → the run id and the messages change only for `pull`. Correct, and anchor behavior is unchanged.

**Findings**

1. LOW, `design`, `internal/fabricengine/hubpull.go:137`.
   - With `SkipGit` set, `Fabric.Pull` returns an empty `PullResult` (`pull.go:278`), so `WeftPulled` is false.
   - Every pair side is then reported `stopped` with "records side's origin could not be reached", while the board side reports `up_to_date`.
   - The verb exits non-zero with a misleading detail.
   - The fix is to treat `SkipGit` as up to date in `pullPairSide`, as `pullBoardSide` already does.
   - I did not confirm that `EnvSyncOptions` can set `SkipGit` in production.

2. LOW, `design`, `internal/fabricengine/hubpull.go:162-176`.
   - In `localBeamLocations`, a single unreadable origin record or failed `NamedWorktree` returns an error for the whole list.
   - `PullHub` then returns early, so every later beam is never attempted. That contradicts "a stop on one side never skips the next".
   - The fix is to report that slug as a stopped beam side and continue.

No other defects found in my lens.

**Tool calls**
1. `grep` of `lymcwd.go` and `sed` of `pull.go:266-330`: to confirm what `Location.WorktreePath()` points at and how `Fabric.Pull` handles `SkipGit`, neither of which the manifest showed.
