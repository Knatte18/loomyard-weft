```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes through one concrete case.

**Per function and branch: the case I ran, and the verdict**

- **`leftoverKind`** (`internal/dobbyengine/leftover.go`):
  - Landed run, pair present, nothing covering it → `landed`. Correct.
  - Cancelled run with keep and its entry ended → none. Correct.
  - Cancelled run without keep → `cancelled`. Correct.
  - Handed-over run under a hold and a pause → `handed_over`. Correct.
  - Handed-over run adopted back (`OwnedElsewhere` false, mark dropped at the takeover write) → falls through to the other kinds. Correct.
  - Pair absent, pass in flight, or driver lock held → none. Correct.
  - Done run that is not Landed: impossible, because `Derive` puts Landed first for batten `done`. Correct.
- **`keptPair`**: kept run whose pair is gone → false. Correct.
- **`tearDownLeftovers`**:
  - Teardown in flight → returns before reading orphans. Correct.
  - Read result, then rebuild the list over `listedRuns` (done runs included) and the due orphans. Correct.
  - Retry map keyed by slug across kinds. Correct.
  - Empty due list → no call. Correct.
- **`readLeftoverTeardown`**:
  - Nil result for an orphan → drops its first sight and its due entry. Correct.
  - Map with the slug missing → `errTeardownUnreported`. Correct.
- **`markKeptCancels`**:
  - Nil seam → marked. Correct.
  - Failure, then success → logs once, then marks. Correct for the engine.
  - Write succeeds but the commit fails: not correct, see F1.
- **`noticeKeptPairs`**:
  - Fresh start, marking done, one kept pair → one notice. Correct.
  - Re-exec → no notice. Correct.
  - No kept pair → nothing posted, and the notice owed is cleared. Correct.
- **`dueOrphans`**:
  - Nil seam or empty record path → nil. Correct.
  - First sight → record written, notice posted, not due. Correct.
  - After a restart → the first sight from the record is kept. Correct.
  - Listing error → cached list returned. Correct.
  - Notice post fails: not correct, see F2.
  - Commit inside the grace (`LastCommit` = now−1h) → not due. Correct.
  - Zero `LastCommit` → due once past the grace. Correct.
- **`dropOrphanSight`**: write fails → Warn, and the next read drops the slug. Correct.
- **`updateOrphanSights`**: `state.UpdateJSON` calls the mutation once under the lock, so `fresh` is never doubled. Correct.
- **`clearDoneRuns`** with `PairPresent`: a landed pair blocked by an owner elsewhere is never cleared, which matches the intent. Correct.
- **`BuildView`** Kept list: kept pair standing → listed; kept pair gone → not listed. Correct.
- **`DobbyRuns`** additions:
  - `keptSince` on a `Finish.EndedAt` that does not parse → zero. Correct.
  - `ReadHandOverMark` error → the whole read fails, matching the other reads. Correct.
- **`MarkKeptCancels`** (battencli):
  - Cancelled record, no intent, pair stands → write, commit, push. Correct.
  - Commit refused (records merge in progress) after the write: not correct, see F1.
- **`DobbyOrphans`**:
  - Prime and prunable worktrees skipped; a worktree that does not resolve → Warn and skip; on the board → skipped; batten run directory present → skipped. Correct.
  - `PairLastCommit` fails for one pair: the whole listing fails, see F4.
- **`PairLastCommit`**: neither worktree present → zero. Correct.
  - A repository with no commit: `CurrentSHA` errors, which feeds F4.
- **`tearDownLeftovers`/`tearDownLeftover`** (beamcli):
  - Ended pairs go to the reconcile; the others one by one; then the workspace is regenerated, with a Warn on failure. Correct.
  - Landed → `Run` with `Remote` and `Landed` set, so the claim goes done. Correct.
  - Cancelled → `Run` with `Force`, and `Run` removes the cancelled record. Correct.
- **`RemoveHandedOverPair`**: the hand-over was made from inside the pair, so its session is live. Not correct, see F3.
- **`RemoveOrphanPair`**:
  - Archive fails → nothing removed. Correct.
  - Uncommitted files → committed off the tip and tagged. Correct.
  - `Run` with `LeaveClaim` and `Force` → session ended, then the pair removed. Correct.
  - Stale due list: see F5.
- **`WriteKeptCancel`**:
  - Cancelled record with no intent → written. Correct.
  - Record already holding an intent → `false, nil`. Correct.
  - Absent record → `false, nil`. Correct.
- **`PairPushDeferred`** and `pairsToPush`:
  - Pending flag present → pushed; absent → skipped; nil seam → old behaviour. Correct.
  - A push in flight also holds a flag (`coalesceFlagged` marks owed sides pending), so a lift may spawn a duplicate push. That push coalesces, so it is harmless. Correct.
- **`DobbyLines`** kept line: zero `Since` → "age unknown". Correct.

**Findings**

- **F1. MEDIUM, design.** `internal/battencli/leftoverpairs.go:56-66`, `MarkKeptCancels`.
  - Summary: a kept-cancel intent that is written but whose commit fails is never committed or pushed through the synchronous path.
  - Trace: `WriteKeptCancel` writes `Cancel` onto the local record. `commitCancelRecord` then refuses, for example on a records merge in progress, or its `Commit` errors. On the next tick the guard at line 38 (`run.status.Cancel != nil`) skips the run, and the dobby still counts the marking as done.
  - Effect: the record stays uncommitted locally until some unrelated commit sweeps it up, against D7's "pushed through D1's synchronous path".
  - Evidence: `leftoverpairs.go:38`, `leftoverpairs.go:56-66`, `cancelcmd.go` `commitCancelRecord` (merge-active refusal).
- **F2. MEDIUM, design.** `internal/dobbyengine/orphan.go:635-661`, `dueOrphans`.
  - Summary: the first sight is recorded before the notice, and a failed post is only logged.
  - Trace: `d.post` logs a Warn on failure (`loop.go:453-460`), for example when no orch is live. A dobby crash between the record write and the post has the same effect.
  - Effect: the orphan is never noticed, yet it is still torn down `OrphanGrace` after the unnoticed first sight. D7 asks for teardown "no earlier than the orphan grace after that notice".
  - Evidence: `orphan.go:635-661`, `loop.go:453-460`.
- **F3. MEDIUM, design.** `internal/pairteardown/leftover.go:18-26`, `RemoveHandedOverPair`.
  - Summary: no session end runs before the removal, and `EndSubjectProcesses(prime, slug, false)` ends none of the task's own processes.
  - Trace: `subjectRuns` skips `slug == owner`, and a task's adopt subject is its beam or root, never itself. So the call ends nothing, and `RemovePair` is called directly although its contract says it "runs only after the session end succeeded".
  - Effect: in the case this kind exists for, a hand-over made from inside the pair, that pair's reed session and registered processes are still live when the dobby removes their worktree. This copies `HandOver`'s existing pattern, but the new caller reaches the case where the session is live.
  - Evidence: `leftover.go:18-26`, `endsubject.go:25-47` (`subjectRuns`: "owner itself is never one of its own runs"), `teardown.go:410-411` (RemovePair contract), `handover.go:146-150` (InsidePair leaves the pair).
- **F4. LOW, design.** `internal/battencli/leftoverpairs.go:108-111`, `DobbyOrphans`.
  - Summary: a `PairLastCommit` failure on any one worktree (a repository with no commit, an unreadable tip) fails the whole orphan listing.
  - Effect: one broken pair stops every orphan from being noticed or torn down, logged only once per episode. A worktree that does not resolve is skipped with a Warn, but this failure is not.
  - Evidence: `leftoverpairs.go:96-111`, `fabricengine/orphan.go:33-40`.
- **F5. LOW, design.** `internal/dobbyengine/orphan.go:625-626` with `internal/pairteardown/leftover.go:34-47`.
  - Summary: between probe ticks `dueOrphans` returns the cached due list, and a probe tick that meets a teardown in flight never re-reads it.
  - Trace: `RemoveOrphanPair` checks again neither the board entry, the batten record nor a live process. It runs with `Force` and an ended verdict, so `EndSession` skips the driver checks.
  - Effect: a pair put on the board, or one that starts a process after the last read, can be torn down within that window.
  - Evidence: `orphan.go:625-626`, `leftover.go:345-358` (early return before `dueOrphans`), `pairteardown/leftover.go:46`, `skip.go:30-32`.

Tool calls:
1. One Bash call read `dobbyengine/loop.go` `post`, `pairteardown/handover.go` `HandOver`, `fabricengine/pendingflags.go` (pending-flag writers), `state.UpdateJSON` and `pairteardown` `EndSession`. My context lacked how a post failure is handled, how `HandOver` sequences its removal, who writes pending flags, whether a mutation can run twice, and what `RemovePair` skips without `EndSession`.
