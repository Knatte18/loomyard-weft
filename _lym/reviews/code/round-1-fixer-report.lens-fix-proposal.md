```yaml
lens: fix-proposal
outcome: reported
```

Challenged proposals: F1, F2, F3, F11, F12. One finding, three more sound but with small gaps.

**MEDIUM — F3, `RemoveHandedOverPair` now calls `td.Run` in place of `td.RemovePair`** (`internal/pairteardown/leftover.go`)
- `Run` deletes and commits the batten run record when its state is cancelled (`teardown.go:565`, `followup.go:72-102`).
- The record deletion commits on the prime's records side and spawns a push.
- Case: a run this machine cancelled with keep, then handed over, so the new owner's machine owns it.
  - It has `OwnedElsewhere` and `HandOverMarked`, and the local record still reads cancelled.
  - `RemoveHandedOverPair` then deletes that run's record on the prime and pushes the deletion.
  - That is a board-record write for a run another machine owns.
  - The old `RemovePair` call and the function's doc ("takes no origin step and writes no board") did not do this.
- `Local: true` does not stop it, because `Run` calls `removeCancelledRecord` whenever it is set, whatever the request says.
- A proposal that ends the session without `Run`'s tail would avoid it: call `td.EndSession(ctx, req)` and then `td.RemovePair(req)`.
- The doc comment now claims the removal is local, and it is not for a cancelled record.

**LOW — F2, `dueOrphans` notice before first sight**
- A notice that posts followed by a failed first-sight write re-posts the notice at every probe tick until the write works.
- The old order, record then notice, could not repeat a notice.
- Evidence: `orphan.go`, where `postOrphanNotice` runs before `updateOrphanSights`, and a write failure returns early.

**LOW — F11, `DobbyOrphans` skip on a failed last-commit read**
- It calls `logger.Warn` per worktree, per probe tick, with no `logger.EpisodeSet`.
- A persistently broken pair logs a Warn every ~30 s.
- The sibling unresolved-worktree branch has the same shape, so this is a consistency gap rather than a regression.

**LOW — F1, `MarkKeptCancels` retry selection**
- `MarkKeptCancels` now calls `commitCancelRecord` for every standing cancelled run with a finished non-remove intent by this machine. That includes runs whose cancel committed long ago.
- Each is a git commit attempt that commits nothing, and `MarkKeptCancels` runs on every dobby start.
- It selects on "intent finished by this machine", not on "records HEAD lacks it". An owed push for an already-committed record is therefore never retried here, since `committed` is false.
- The reviewer's wording ("while the prime's records HEAD lacks that status change") was only partly carried out.

**F12:** no case found.
- `requireStillOrphan` reads the local Bolt board, the same source `DobbyOrphans` reads, so the two do not drift.

Further tool calls (2):
1. `grep` for `removeCancelledRecord` in `internal/pairteardown`, to see whether `Run` calls it for every request. My context did not show `teardown.go`'s struct and wiring.
2. Read `followup.go:64-102`, to see what the deletion writes and pushes. My context held only `Run`'s call site.
