```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each changed function and error path with a concrete input.

**List: function or branch, case run, verdict**

- `statuscommit.New` and `waitForOperation`:
  - One-repo deps with `MergeActive` nil, a probe returning "cherry-pick" and then "": it waits, commits, pushes. Correct.
  - Commit failure with `Stalled` true: it waits and commits again. Correct.
  - Protected push failure: it returns nil. Correct.
- `failureDisposition`: re-probe error, operation present, and Stalled each wait. A permanent commit failure with a failing probe waits forever, which the doc states. Correct.
- `retryHeldIndexLock` with a 350ms bound: sleeps 50, 100 and 200, makes 4 calls, returns the last error. Correct.
- `oneRepositoryAt`: records sibling with owner beside it, prime on a branch without the marker (falls to the `lym-bolt` marker), Bolt checkout, non-repo path. Correct.
- `refuseOffPrimeBranch`: key absent (exit 1), detached HEAD, task worktree, two-repo prime. Correct.
- `lymcwd.OneRepository` / `readRecordedAnchor`: blank root marker gives ".", invalid marker is an error naming the root file, sibling and Bolt are excluded. Correct for the cases run. Finding 4 below.
- `RecordsPathsAt`, `IsRecordsOnlyCommit`, `IsFabricMergeCommit`: root commit, marker-deleting commit, 3+ parents, `merge-base` exit 1, `merge-tree` conflict. Correct.
- `refuseSourceMarkerChange`: reverted edit, anchor-change trailer, newest hand edit after a trailered change, fabric merge commit. Correct.
- `settleSourceRecords` and `restoreRecordsToHead`: source-only file, records conflict, write between steps 2 and 3, squash with nothing left staged. Correct.
- `Merge` guard order: the in-progress guard runs before the sync step, which runs before `commitReceivingRecords`. Correct, no commit lands around a parked merge.
- `FastForwardPair` one-repo: `localHEAD` is read after the records commit. Correct.
- `commitRecordsAroundMergeWith`:
  - Plain merge, unsettled merge, cherry-pick, no merge: correct.
  - Failure after `update-ref`: see finding 3.
- `holdPrimePushForOperator` and `primePushOperatorCommits`: operator commit, merge trailer on a code commit, fast-forwarded side branch, fabric merge with its second parent held by origin, standing protected record. Correct.
- `classifyFailure`, `IsPushProtected`, `IsPushOwed`: correct. Retry change in finding 2.
- `replayWeft` one-repo (`rewordWarpSHA`, `cherryPickReworded`): trailer naming a replayed commit, one naming another commit, a code commit stops the replay, a dropped empty pick skips the reword. Correct.
- `CloneHub` / `cloneOneRepository` / `PushOneRepositoryClone`:
  - Second URL, origin has `lym-bolt`, binding present, missing marker, refused push all tear the hub down.
  - After a refused board push, origin's `main` keeps the marker commit, as D2 says.
  - A re-run over that origin takes the `HasAnchor` branch. `commitAnchorMarker` is a no-op when the marker is unchanged (I read it), so the re-run works. Correct.
- `ensureBoardWorktree` tracking branch: local absent and remote present, local present, neither. Correct.
- `Add` one-repo: fork, adopt, rollback, run-records drop. Correct, except finding 1.
- `removePair` / `runRemoteTeardown` one-repo: view ref gone, archive skipped, lease rejected. Correct.
- `takeoverBranch.land`: write lock taken before the push lock, released before the push, guard hold drops the entry commit. Correct.
- `mergeCommitMessage` / `preparedMergeMessage`: explicit message, `MERGE_MSG`, `SQUASH_MSG`, none. Correct.
- `MergeStateOperation` / `MergeStateActive`: one-repo parked fabric merge reads "". `tendercli/runintents.go:240` therefore no longer refuses on a parked fabric merge in a one-repo prime. That follows D4 (a records commit never skips), so it is not a defect.

**Findings**

1. MEDIUM, design. `internal/fabricengine/add.go`, the one-repo `anchorTo` branch (diff lines 224-229). `Add` with `opts.Anchor` only calls `stageNewMarker`, so tracked records at the fork's anchor (such as `_lym/config/*`) are never moved to the new anchor. Two-repo `moveForkedWeftToAnchor` moves them, and D5 says an anchor change moves `_lym` with the marker. A task added at another anchor starts without the parent's tracked records, and the old anchor keeps them. The tests only cover anchors where `_lym` holds nothing but the origin record. I did not read `stageNewMarker`, so this is read from the call site and comment only.

2. LOW, design. `internal/fabricengine/pushclassified.go`, `rejectionRetryKind` (diff line 138). `ProtectedBranchRefusal(err)` now ends the rebase-retry for every branch. D7 says other branches keep today's treatment, and only the class is tested (`ProtectedRefusalOfATaskBranchKeepsTodaysClass`). A protected refusal that arrives as `ErrPushRejected` on a task branch used to be retried and is now returned at once.

3. LOW, design. `internal/fabricengine/recordsduringmerge.go`, `commitRecordsAroundMergeWith`, after `update-ref` (diff lines 306-319). If `newPaired` or `saveMergeState` fails, it returns `(sha, true, err)` with HEAD moved but `WarpStart` stale. A later `MergeAbort` resets to the old start and drops the records commit.

4. LOW, design. `internal/lymcwd/anchor.go` `OneRepository` reads the working tree. Fabric's `oneRepositoryAt` reads git and falls back to `lym-bolt`. A one-repo prime switched to a branch without the marker reads as two-repo in `lymcwd`, so anchor resolution falls back to "." for a subpath hub, while fabric still reads it as one-repo. The commit is refused, but anything written at the wrong anchor before it stays on disk.

Outside my lens: `pattern/PATTERN-run-fsm.md` says a `protected` push "stays owed". D7 and `IsPushOwed` say it is not owed.

**Tool calls**
- Bash (`grep` `commitAnchorMarker`, `MergeStateActive` callers, `Merge` line): `anchormarker.go` and the callers were not in the diff.
- Read `tendercli/runintents.go:225-255`: to check whether a caller refuses on a parked merge.
- Read `fabricengine/merge.go:373-497`: to confirm the guard-before-commit order in `Merge`.
- Read `fabricengine/parenttips.go:340-435`: to check where `localHEAD` is read in `FastForwardPair`.
