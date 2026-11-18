```yaml
lens: code-scope
outcome: reported
```

Lens: code-scope. I traced each change in the diff to a goal point of the board entry under `## Goal`; the record has no `Added after Discussion` entries.

**Decisions (from the Goal)**
- **G1a:** a pause hold never waits while holding a shared lock.
- **G1b:** dobby starts no batten driver for a paused pair.
- **G2:** pause and status record a pair with no reed session, report it under skipped, and go on.
- **DW1, DW2, DW3:** the three Done-when tests.

**Per change → decision**
1. **Gate before the push lock.**
   - Code: `pauserecord.Admit(pushLockGateOp)` before each push-lock taker: `coalescePush`, `coalesceFlagged`, `lockWeftForReplay`, `retryBoltPush`, `takeoverBranch.land` and `Bolt.pullFirst`.
   - Docs: the `pushLockGateOp` const and the `doc.go` paragraph at 1241ff.
   - Traces to G1a.
2. **Deferral simplified.**
   - Code: the `pushed` map is removed, `pushDeferral` puts back every owed flag, and the `doc.go` lines at 1302ff and the `coalescePushSides` doc are updated.
   - Forced by change 1: with the gate before the lock, no side has been pushed when the deferral fires.
3. **Fabric tests.**
   - `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock` is new: traces to DW1.
   - The rewrite of `TestCoalescePushRequest_CoveredCallDefers` and `TestPushDeferral_PutsBackEveryOwedFlag`, and the helpers moved to file scope, are forced by changes 1 and 2.
4. **`boardcli/syncdeferral_test.go` row `PlainCoveredWaitsThenReachesTheBoard`.** Forced by change 1: a plain covered sync now waits at the gate before the board's push lock.
5. **Pause and status skip a pair with no reed session.**
   - Code: `coveredSessions` and `sessionStatuses` match `reedengine.ErrNoSession`, `noReedSessionNote` is added, and `StatusReport.Skipped` is new.
   - Docs: the `Pair.Sessions` doc, the `tenderengine/doc.go` lines and the `tendercli` help text.
   - Traces to G2.
6. **Tender tests.** The new `TestPause` row and the new `TestStatus` subtest trace to DW3.
7. **`Revive` reports a pair with no reed session under `ReviveReport.Skipped`.**
   - Forced by change 5: `Revive` shares `coveredSessions`, which now swallows `ErrNoSession` for every caller.
   - Revive therefore either says why the pair was skipped or drops it silently, and reporting it is the minimal honest handling.
   - This is not unrequested scope.

**Per decision → changes**
- G1a: change 1, with changes 2 to 4 forced by it.
- G1b: **no change.**
- G2: changes 5 and 7.
- DW1: `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock`.
- DW2: **no test added**; the diff relies on the existing `TestPlanPass` `VerdictPauseCovers*GivesNone` rows.
- DW3: the new `TestPause` row and `TestStatus` subtest.

**Findings**

1. **MEDIUM, scope.** Location: G1b / DW2, the dobby side; the diff touches no `internal/dobbyengine` file, and the claim is in `_lym/landing/summary.md` under "Design decisions".
   - Summary: the Goal's point "Dobby starts no batten driver for a paused pair" and its Done-when test are built by no change. The summary's reason for needing none does not match the code.
   - The summary blames the incident on the pause failing at the missing reed session.
   - But `Pause` writes its live entry with `pauserecord.Update` before it calls `coveredSessions` (`internal/tenderengine/pause.go:146-175`). A pause that failed at the session listing therefore still left the pair covered.
   - Dobby's `PauseCovers` reads that record (`internal/beamcli/dobbydeps.go:335-338`), so the incident's driver start is not explained by the failed pause.
   - The rows the summary cites, `TestPlanPass` `VerdictPauseCovers*GivesNone` (`internal/dobbyengine/passes_test.go:61-87`), pin `planPass` on a run whose listing verdict already waits. They do not pin `startReady`'s first start of a covered task.
   - That start is `readyToStart(task, here, covered)`, which excludes a covered task only through `heldByEntry` (`internal/dobbyengine/starts.go:77-78`, `126-136`). My grep of the dobby tests found no test that sets `covers` and asserts no first start.
   - Fix: either add a test, or make the change, showing dobby's start path starts no driver for a pair a live entry covers. Otherwise the gap goes to the parent as a recorded decision; the summary's causal claim alone does not settle it.

No other change lacks a decision, and no other decision lacks a change.

Tool calls:
1. `grep` of the dobby tests for `covers`/`Covered`, `dobbydeps.go` 325-345 and `readyToStart` in `starts.go`. My context had the `planPass` rows but not the start path, its test coverage or how dobby reads coverage.
