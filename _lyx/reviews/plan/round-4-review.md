---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-battenshed-read-guards.md
    summary: Card 9's three ReadDecision guard rows cannot reach their sites or recovery, because the awaiting arm's own ReadDecision error hard-ends the Call on the same check, and the handed-lock decision read never repeats at all
    fix: "In 09-battenshed-read-guards.md, remove the Edit target `internal/battenshed#innerRunProducer.handedLockStep`, remove handedLockStep's decision read from the list of guarded sites in Intent, and remove the guard row 'the decision read for a handed lock'. In the 00-overview.md D11 table, take `handedLockStep` out of the battenshed guard row's site list and add a row: site `battenshed` `innerRunProducer.handedLockStep`, line `decision unreadable for the parent lock step; keeping the handed lock`, verdict `keep: a failed decision read ends the awaiting arm's Call on the same check through awaitingStep's hard end, and a pause-covered check returns from lockStep before the read, so the line never repeats`, card `—`. In card 9's TestInnerRun_ReadFailuresLogOncePerEpisode, run the stale-base decision-read row and the notice step's decision-read row with `InnerRunDeps.Pause` answering a coverage that covers the slug on every check, so runArm skips awaitingStep while noticeStep still runs. Their one ReadDecision seam is read by parentMovedUnderGate and by undecided on the same probe check, so it fails on its first six calls and then answers, and each row asserts its own message's one WARN, at least one DEBUG and one INFO `a repeated failure recovered` line naming it. Add a sentence to Intent saying these two sites repeat only while a pause covers the slug, since otherwise awaitingStep's own decision read ends the Call."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-battenshed-read-guards.md
    summary: The TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn extension cannot reach the strand read's recovery, because the test probes once and its ReviveStrands stub fails the test on any revive
    fix: "In 09-battenshed-read-guards.md, in the sentence extending TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn, add that the test sets `deps.NoticeProbe = 0` and `clock.pauseAtSleep = 4`, so the halted arm calls reviveDeadDriver on the entry and on every check before the pause ends the wait, and that DriverStrand answers `ChildDriverLive` after its three failures, so the existing ReviveStrands stub, which fails the test on any revive, is never called."
---

### [BLOCKING] Card 9's decision-read guard rows are unreachable, and one guarded site never repeats

**Location:** `_lyx/plan/09-battenshed-read-guards.md`, Intent guard rows; `_lyx/plan/00-overview.md` D11 table, battenshed guard row.

**Issue:** All three `ReadDecision` sites the card guards (`handedLockStep` through `resumePending`, `parentMovedUnderGate`, `undecided`) read `w.status.State == StateAwaiting`.
In this code that state only occurs in a Call whose arm is `awaitingStep` (`innerrun.go:426`), because `checkChild` ends the wait whenever the state differs from the one the Call began with.
`awaitingStep` (`innerrun.go:806`) calls `p.deps.ReadDecision()` itself and returns `hardEnd` on any error.
Here is the order in `wait` (`innerrun.go:579`): on entry, `noticeStep` → `noticeCondition` → `parentMovedUnderGate` (with `baseStale` true) → `resumePending` → `ReadDecision` fails and logs the stale-base Warn.
Next, `runArm` → `awaitingStep` → `staleBaseResume` → `parentMovedUnderGate` returns the cached `baseMoved` → `awaitingStep`'s own `ReadDecision` fails → `hardEnd`.
With the card's seam failing on its first three calls, every one of these rows ends the Call at entry.
The handed-lock row never reaches `lockStep`, because `lockStep` runs only in `check`, never at entry.
The notice-step row never reaches `undecided`, and the stale-base row logs one Warn and no DEBUG or INFO line.
None can pass its assertions.

The production verdict is affected too.
On an uncovered check, `lockStep` → `handedLockStep` runs before `runArm`, so a failing read logs once and then `awaitingStep` hard-ends the same check.
On a pause-covered check, `lockStep` returns before reading (`innerrun_parentlock.go:37`).
So the handed-lock decision Warn cannot repeat without a state change, and its D11 verdict is keep, not guard.
The stale-base and notice-step reads do repeat while a pause covers the slug, because `runArm` skips the arm (`innerrun_pause.go:39`) but `check` still calls `noticeStep`.
Their rows therefore need the pause coverage to reach the site at all.
They also need enough failures that each of the two sites sharing the one seam per probe check logs a DEBUG line before the seam heals.
Checked with `lyx loom validate-plan --overlay`: card 9 without the `handedLockStep` target reports no finding the `{}` baseline does not.

### [MEDIUM] The extended strand-read test probes once, so recovery is never reached

**Location:** `_lyx/plan/09-battenshed-read-guards.md`, the sentence extending `TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn`.

**Issue:** The existing test (`innerrun_halted_test.go:284`) builds its deps with `newInnerRunDeps`, which sets `NoticeProbe: time.Hour` and `clock.pauseAtSleep = 1`.
`haltedStep` calls `reviveDeadDriver` only when `w.probeDue` is set (`innerrun.go:708`).
So the Call probes once, at entry, and ends at its first check.
A `DriverStrand` that fails on its first three probes is therefore called once, which logs the WARN but no DEBUG line and no recovery INFO, and the card's new assertions fail.
The card also does not say what `DriverStrand` answers after it heals.
`ChildDriverDead` would call `ReviveStrands`, whose stub in this test calls `t.Error`.
The card has to name the probe cadence, the wait length and a non-dead post-heal answer.

## Settled focus questions

- Per-iteration call graph: I followed the reed watch tick's state load (`loadOrInitStateLocked`, `adoptPaneGenerationLocked`, `refuseLiveForeignSessionLocked`, `LoadState`, `requireSessionLocked`, `withTryOpLock`, `strandWindowTargetFor`, `hookInstalledLocked` at Debug, `planLayout`, `fixedHeightPins`) and the name-repair pass.
  I also followed shuttle's readers reached through loomcli's and battencli's callbacks (`ReadAgentWaits`, `heldMarkerLive`, `ReadAgentActivity`, `newestTurnEnd`, `ReadSessionStates`, `readRunSessionState`, `ReadWaitMarkers`, claudeengine's reader paths), loomcli's `idleReleaseOn` and `parentlock.Store.List`.
  Every Info-or-above line I found on them has a row in the D11 table.
  `installResizePinsLocked` is not reached from the watch tick, because `reapplyLayout` passes `SkipFocus` without `InstallPins`.
- The keep verdict on `parentlock` `Store.withLock`'s release Warn holds: gofrs/flock v0.8.1's `Unlock` returns on a failed `LOCK_UN` (Unix) or `UnlockFileEx` (Windows) before closing the handle, so the lock stays held and the next blocking `Lock` waits instead of logging again.
- The keep verdict on `battencli` `readSessionStatesByStrand` holds: `ReadSessionStates` errors only on the run root's `ReadDir`, which `ReadAgentActivity` has just read on the same root within the same `readAgentActivity` call.
- `repairSessionName`'s `repaired session name` and `repairNames`' `repaired pane title` stay keep.
  Each line records an action reed just took, a typed rename or a `select-pane -T` under `allow-set-title off`, so the line is once per event even on a rare path where the drift returns.
  `runResizePinsLocked`'s per-pane guard covers the fallback-box path, where the box-equality skip is never taken.
- Recovery reach in cards 9–15: card 9's new rows reach their sites except the decision reads in F1, and the extended `TestInnerRun_RunningArmReviewWaitNote` reaches `reviewWaitNote`'s recovery.
  That test sets `NoticeProbe = 0` and `pauseAtSleep = 3`, so `ReviewWait` is called at entry and on each check, and its seam heals by the third check.
  The strand-read extension does not reach recovery (F2).
  Cards 11, 13, 14 and 15 each name a test row that fails the seam and then heals it, and the healing input reaches the recovery branch: `ReadLoopPIDRecord` reads a missing file as no error, `Store.List` reads a restored `Dir`, and the queue-driven fakes answer after their failures.
  In card 12, `TestRun_SuccessfulTickResetsErrorCount` and the network-markers row reach their recovery lines.
