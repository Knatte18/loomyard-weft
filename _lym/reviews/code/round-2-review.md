---
verdict: APPROVED
---

No finding survived judgment against the decision record's three faults and the repository's rules.
The four focus questions are settled below.

## Focus verdicts

- **`DropLocal`'s untracked-file removal.**
  `TestDropLocal` shows each case.
  - An untracked weft file origin's tip tracks is removed, the weft moves to origin's tip, and the recorded kinds are `tag_created`, `path_removed`, `commits_dropped`.
  - An ignored file on a path origin tracks is overwritten by git and not removed.
  - A weft file origin's tip does not track survives with its content and no `path_removed` entry.
  - An untracked code-branch file origin tracks is never removed: `untrackedOnPaths` runs only for the records side, and git refuses the move with nothing moved.
  - `TestOwnedUntrackedTrackedAt_RefusesATargetTheIndexTracksOrTheCommitDoesNot` covers the gate's three outcomes: owned, refused as index-tracked, refused as not tracked at the commit.
  - The two refusals cannot be reached through `DropLocal`, because `untrackedOnPaths` already filters to untracked paths in the changed set, so the direct test is the only coverage and is not redundant.
- **The reconcile seam.**
  `TestReconcileSeamWaitsOnAPushLock` fails if `reconcileSeam` stops passing `reconcilePushLockWait` as `PushLockWait`, or passes a non-positive bound.
  The one-line call to `reconcileSeam` inside `hubDepsAs` is not pinned by any test, and I judge that not a defect: it needs a full hub fixture for one line of wiring.
- **Docs.**
  The `doc.go` Step done section, the Resume bullet, the exemption paragraph and the `docs/overview.md` adopt entry all agree with `rejudges`, `defer deps.ExemptPause()()` in `runAdopt` and `runWorker`, and `waitRecordsLocks`.
  The old "only the local-commit flag differs" sentence is replaced, and no stale resume claim remains.
- **Busy-at-once and kept paused slugs.**
  `TestReconcileOrigin_Moves` shows a held write lock reported busy on `weft write` with zero poll sleeps, nothing moved, under a nonzero `PushLockWait`.
  It also covers the push lock released mid-wait (moved) and held past the bound (busy after three sleeps), on an injected `Sleep` with no real time.
  `TestRunWorker` shows a re-judged stop that still holds ending stopped again with the same held-back set and way forward.
  It also shows a re-judge keeping the recorded paused slugs without pausing them again.

## Rejected

- Pushing the untracked file's removal ahead of `ResetKeep` loses the file's local content if the reset then fails, since the archive tag keeps only committed state. The decision record asks for exactly this removal, and the error names the tag, so it is no finding.
- `waitRecordsLocks` can wait up to the bound once per records branch in series. The bound is short and applies only while a push lock is held, so this is not a defect.
- `untrackedOnPaths` uses `gitexec.Run` directly. `gitexec` is the sanctioned client path, and neighbouring `fabricengine` code does the same.
