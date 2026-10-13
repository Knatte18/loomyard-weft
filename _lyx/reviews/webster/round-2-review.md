---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: cmd/lyx/testdata/test-budget.yaml:108
    summary: "TestTestBudget_PackagesStayWithinBudget fails because internal/shuttleengine/claudeengine holds 110 top-level tests against a budget row of 109"
    fix: "In cmd/lyx/testdata/test-budget.yaml change the row for internal/shuttleengine/claudeengine from 109 to 110, then run lyx gate test ./cmd/lyx to confirm the budget test passes."
---

### [BLOCKING] Test budget row for claudeengine is one short after the merge-in

**Location:** cmd/lyx/testdata/test-budget.yaml:108
**Issue:** The Publish verify failed on `TestTestBudget_PackagesStayWithinBudget` (`.lyx/verify/verify-3.log`): `internal/shuttleengine/claudeengine: 110 top-level tests, over its budget of 109`.
The merge-in commit 5ea9b0721 brought `TestInterruptGuardReadings` into `internal/shuttleengine/claudeengine` from the parent, and this branch's own `TestOpenLaunch` in `command_test.go` was already counted in the row.
The row on the branch is therefore one short of the package's real count, and the test names the fix itself: the edit lands in the commit that adds the tests.
No other package is reported over budget in the log.

## Verdicts on the focus questions

- `launchStrandLocked` relaunch (`internal/reedengine/spawn.go:84`, `killReplacedOwnWindowPaneLocked`): not a defect.
  `previousPaneID` is read before the new pane exists, and the kill runs only after `createOwnWindowPaneLocked` has bound the new pane, so a shared window holding only the old pane is never ended mid-launch.
  `planReconcile` keeps a present own-window pane's binding, alive or dead, so the strand still names its old pane when launch starts, and `ownWindowsHoldingNoStrand` does not end that window before the new pane joins it.
  `groupLiveCount` excludes the relaunching strand itself, so the plan joins the group window and the group ends with one pane per strand.
  Resume and `add --if-absent` both reach this one chokepoint, so the kill covers both; the integration test drives Resume only, and that is enough for one shared code path.
- Placement constants (`internal/reedengine/ownwindow.go:106`): not a defect.
  `placeApart = iota`, `placeNewShared` and `placeJoinShared` evaluate to 0, 1, 2, the values the explicit literals held, so no placement decision changes.
- `TestOwnWindow_RealTmux`: not a defect.
  `RelaunchOfADeadGroupMemberRejoinsItsWindow` pins a dead shared-group member relaunched into its group window with the old pane gone and two panes left, and `RelaunchRejoinsAWindowKeptByItsDeadPaneAndTheLastRemovalEndsIt` pins the window kept alive by the dead pane through the relaunch, which no other test covers.
  The second case's removal tail overlaps `RemoveEndsTheOwnWindow` only in its last assertion; it is not a duplicate behavior.
