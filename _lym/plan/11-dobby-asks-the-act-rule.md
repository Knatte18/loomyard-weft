# Card 11 — dobby-asks-the-act-rule

**Create:**
- `plan:internal/dobbyengine#TestDrivePasses_HandOverRecordsOwnedElsewhere` -> `func TestDrivePasses_HandOverRecordsOwnedElsewhere(t *testing.T)`
- `plan:internal/dobbyengine#TestSlotsTaken_CancelledRunTakesNoSlot` -> `func TestSlotsTaken_CancelledRunTakesNoSlot(t *testing.T)`
- `plan:internal/battencli#TestDobbyRuns_CarriesTheActAnswer` -> `func TestDobbyRuns_CarriesTheActAnswer(t *testing.T)`

**Edit:**
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#slotsTaken`
- `internal/dobbyengine#strandedEnded`
- `internal/dobbyengine#daemon.readWaits`
- `internal/dobbyengine#daemon.runNotices`
- `internal/dobbyengine#dobbyengineAllowedImports`
- `internal/dobbyengine#TestPlanPass`
- `internal/dobbyengine#TestStrandedEnded`
- `internal/dobbyengine#TestRunNotice_Conditions`
- `internal/dobbyengine#TestWaitReminders`
- `internal/battencli#DobbyRuns`

**Uses:**
- `internal/dobbyengine/doc.go`
- `internal/dobbyengine#daemon.ownedElsewhere`
- `plan:internal/runfsm#State`
- `plan:internal/runfsm#Verdict`
- `plan:internal/pairteardown#AskAct`

**ImpactSummary:** dobby gives a run a pass only when the act rule answers act, hands a run another machine owns over without a pass, and counts no slot for a cancelled run.

**Intent:**
Make the dobby's pass ask the act rule first (D1).

`RunView` drops `CancelMarker` and gains `Phase runfsm.State` and `Act runfsm.Verdict`.
`DobbyRuns` fills both through `pairteardown.AskAct` for every listed run, after it reads whether a paused run's work reached its parent.
It sets the shed pause aside for such a run only, so a paused run whose work already reached its parent is still driven to its landing, as `drivable` allows today.
`dobbyengine` gains `internal/runfsm` in `dobbyengineAllowedImports`.

`planPass` gives no pass unless the verdict is act, before any trigger is read.
Because the act rule waits while a pause covers the run, the continue pass under a covering pause goes;
the lift of a pause still gives the full pass, and the origin-owed marker still makes it a full pass that leaves the reconcile owed.
`daemon.drivePasses`, on a hand-over of a run not yet recorded owned elsewhere, runs `daemon.ownedElsewhere` for it in place of spawning a pass whose exit reports the owner.
A run whose verdict is not act drops a pending lift answer, as a run that cannot be given a pass does today.
`strandedEnded`, `daemon.readWaits` and `daemon.runNotices` read a cancelling `Phase` where they read the cancel marker.
`slotsTaken` counts no slot for a cancelled run, whose board entry is back to draft and whose record stays until its pair goes.

`TestPlanPass` gains rows for a wait and a hand-over verdict giving none and for no continue pass under a covering pause.
The three other listed tests set `Phase` where they set the marker.
`TestDrivePasses_HandOverRecordsOwnedElsewhere` (tier 0, the daemon's fakes) asserts a hand-over spawns no pass and records the run owned elsewhere once.
`TestSlotsTaken_CancelledRunTakesNoSlot` is tier 0.
`TestDobbyRuns_CarriesTheActAnswer` (tier 1, hubforge) asserts a run owned by another machine reads hand_over, a run with a pause intent reads wait, and a halted run of this machine reads act.

**Commit:** 11: dobby passes only a run the act rule lets act
