```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced the changed fabric, landingshed, pairteardown, statuscommit, runfsm and intent-push paths against concrete inputs. I found six defects; the case-by-case list follows them.

## Findings

**C1 — MEDIUM, design.** `internal/landingshed/finalize.go` (`pushParent`, tail where `fz.markTaskDone()` is called and its error returned plain).
- A failed board `done` write is returned as an unmarked error. The `Deps.MarkTaskDone` doc says the driver re-steps the landing and writes the mark again.
- `pushRecords` marks its owed case `shedengine.MarkTransient(TransientGitTransport, …)` so the driver re-steps. `markTaskDone` marks nothing.
- With origin unreachable, the synchronous Bolt `done` push fails with a plain error, which can read as a hard failure instead of a re-step.
- Evidence: `finalize.go`, `markTaskDone` and `pushRecords` in the diff. I did not read how the driver treats an unmarked error, so confirm that before settling severity.

**C2 — MEDIUM, design.** `internal/landingshed/finalize.go` (`landUnderLock` calls `fz.recordLanding(parentHandle)` right after the merge; `pushParent` later calls `archiveHeldParentMerge`).
- `RecordLanding` stores the local merge commit as the run's landing commit before the parent push.
- If the push is then rejected because origin already holds the task's work, `ArchiveParentMerge` tags that merge and resets the local parent branch to origin's.
- The recorded landing commit is now archived and not on the parent branch. The `RecordLanding` doc says the dobby names it as the landing.
- Nothing re-records the landing after the reset.

**C3 — MEDIUM, scope.** `internal/fabricengine/handover.go:pushOwedCodeTip` (`originTip == ""` returns `false, nil`).
- Origin has no code branch for the pair, for example because the first code push failed.
- `OriginCodeTip`/`readCodeTipAt` read an absent branch as `Missing`.
- D2 says the owing machine pushes the tip when origin's code branch lacks it. The hand-over pushes nothing and leaves the tip to its archive tag.
- The adopter waits for the 30-minute grace, then returns to the previous transition. The tip is lost where D2 intends it to land.

**C4 — LOW, design.** `internal/pairteardown/handover.go:HandOver` with `internal/fabricengine/takeover.go:takeoverStop`.
- `HandOverTips` writes the hand-over mark before the local removal.
- If `RemovePair` then fails, `HandOver` returns a non-`ErrHandedOver` error.
- The next rejected push reads the mark and returns `ErrHandedOver` without ever running the removal again.
- The pair stays standing with processes ended and tips archived. Only a separate leftover-pair teardown recovers it.

**C5 — LOW, design.** `internal/fabricengine/takeover.go:takeoverStop` and `MachineNames.Holds`.
- `Holds("")` is false, so a takeover entry with an empty `Machine` counts as another machine's.
- It would hand the run over and archive local tips. `Takeover` is read tolerantly, so a record that omits the key reaches this path.

**C6 — LOW, design.** `internal/fabricengine/pushclassified.go:deferral`, `internal/fabricengine/pushanchored.go:IsPushOwed`, `internal/statuscommit/statuscommit.go:retryOwedPush`.
- A records worktree with an uncommitted tracked change records the push as `PushFailureDeferred`. `IsPushOwed` reads that as owed.
- `retryOwedPush` then retries without limit, so the transition never halts on a cause that only a human can clear.
- The error names the cause, but the run is neither halted nor refused.

## Cases run

- **`coalescePushSides` step:** a records side that failed, was rejected or deferred gives `recordsLanded=false` and no code push. A weft with an unborn HEAD gives `true` and the code side goes out. Correct.
- **`pushSideDetached` landed test:** `HasUnpushed` after a nil push catches a deferral and an ended branch. Correct.
- **`PushPairAnchored` step and `pushRecordsSide`:**
  - `*TakeoverError` passes through `pushSide`'s `%w` wrap. `errors.As` in `PushRun` still finds it.
  - A rejected records push returns a `PushFailedError` joined with the withheld code worktree line. Correct.
  - The prime pushes records only. Correct.
- **`teardownLockOwedError`:** a zero wait returns the refusal with no record. A positive wait writes a `teardown_lock` record that `IsPushOwed` reads as owed. A `CurrentBranch` failure falls back to the plain refusal. Correct.
- **`acquireTeardownShared`:** wait 0 gives one try; a positive wait polls at 1s and stops at the deadline. Correct.
- **`IsPushOwed`:** it handles single, wrapped and joined errors, and `ErrPushLockBusy`. Correct.
- **`takeoverStop`:** zero names or an empty slug do nothing; an entry naming a held name does nothing; a mark naming the same machine gives `ErrHandedOver`; otherwise `TakeoverError`. Correct apart from C5.
- **`retryWeftPush`:** the takeover read runs after the fetch and before any replay. A rejection with no entry replays as before. Correct.
- **`HandOverTips`:**
  - It takes the teardown lock exclusive after `PushPairAnchored` released its shared lock.
  - Order is tip push, archive, parent merge, notice, then mark written last. A failure before the mark leaves no mark, so the next rejection hands over again.
  - A nil pair writes the mark with no tags. Correct, except for C3.
- **`pushOwedCodeTip`:**
  - The lease is held at the origin tip. A given-up tip is moved back with `move(originTip, tip)`. A lease rejection is reported.
  - A diverged origin pushes nothing. Correct, except for C3.
- **`resetUnnamedTip`:** it resets only when the named tip is held locally and origin's tip is neither the named tip nor behind it, using the lease. A lost lease returns `false`. Correct.
- **`lastTransitionCommit`, `trailerValue`, `ReturnToPreviousTransition`:** the newest `Run-Transition` match is taken and other Warp-SHA commits are skipped. Fewer than two transitions returns `false`. A missing previous record is an error. Correct.
- **`OriginCodeTip`/`readCodeTipAt`:** an unknown tip object reads as missing through `isAncestorOrEqual`'s `cat-file` guard. Correct.
- **`ReadRun`:** owed-push error and worktree come from the newest record; the code tip fills only when missing. Correct.
- **`statuscommit.New`:** no commit releases the admission. A commit error releases it first. An owed push releases it and retries from 1s up to 1m. A hand-over error returns without retry. Correct, except for C6.
- **`Finalize.pushParent`:** the owner is read before the parent push, before records and before `done`. `PushSkipped` skips the owner read. A transient parent failure re-steps. Correct, except for C1 and C2.
- **`runHandOver`:** a nil hand-over error would be wrapped as `%!w(<nil>)`. It cannot arise for a wired `HandOver` that follows its doc, so I did not file it.
- **`runfsm.Act`, `Refusal` and `Judge`:** `PushOwed` carries its error and worktree. `Aside` clears all three. Correct.
- **`battencli.clearRunPhase`:** an owed push puts the re-run into the way forward. A failed push is a warning. Correct.
- **`commitRunIntent` and `owedIntentPush`:** an owed push prints a line on stderr and returns nil. Any other error is returned. Correct.
- **Unverified:** for `lym tender pause`, the intent push runs from `Record` after the pause entry is on disk (`internal/tenderengine/pause.go:165`).
  - If the tender process has a gate installed, that push could wait at the gate.
  - The run is paused so it would not push; the test `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` is the check. I did not trace it.

## Tool calls

1. Read `WeftWarpSlug` and `EndSubjectProcesses`: my context lacked their definitions.
2. Read the whole of `internal/fabricengine/handover.go`: the diff saved to disk was truncated.
3. Read `isAncestorOrEqual` and the diffs of tender, batten cancel and commit-records: they were outside the manifest.
4. Read the `ReturnToPreviousTransition` body and the `loomcli/wiring.go` diff: the preview was truncated.
5. Grepped tendercli, tenderengine and `pauserecord.Admit` for the gate and Record ordering: not in context.
6. Read `tenderengine/pause.go` Record ordering and the diffstat of tenderengine and pauserecord: to confirm when the intent push runs.
