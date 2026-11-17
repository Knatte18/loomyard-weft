# Card 8 — The adopt check's ends work through the hold

**Create:**
- `internal/adoptcli#holdCheckDeps`
- `internal/adoptcli#holdCheckResult`
- `internal/adoptcli#runHoldCheck`
- `internal/adoptcli#hubHoldCheckDeps`
- `internal/adoptcli#TestRunHoldCheck`
- `internal/adoptcli#TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry`
- `internal/adoptcli#TestHubHoldCheckDeps_FilledSeamsNameTheRerun`
- `internal/orchcli#DobbyProgram`

**Edit:**
- `internal/adoptcli#checkDeps`
- `internal/adoptcli#runCheck`
- `internal/adoptcli#hubCheckDeps` -> `func hubCheckDeps(prime *lymcwd.Location, subject string, terminal io.Writer) (checkDeps, error)`
- `internal/adoptcli#resolveSubjectWorktree`
- `internal/adoptcli#checkCmd`
- `internal/adoptcli#checkView`
- `internal/adoptcli#fixDeps`
- `internal/adoptcli#failedEnd` -> `func failedEnd(deps fixDeps, record CheckRecord) (fixTask string, lines []string, lost *boardengine.HoldRefusal)`
- `internal/adoptcli#hubFixDeps`
- `internal/adoptcli#fixReadError`
- `internal/adoptcli#fixFilingError`
- `internal/adoptcli#fixBody` -> `func fixBody(subject, rerun string, record CheckRecord, suiteLog string) string`
- `internal/adoptcli#fixRemovalError`
- `internal/adoptcli#greenDeps`
- `internal/adoptcli#greenEnd`
- `internal/adoptcli#removeFixTask`
- `internal/adoptcli#hubGreenDeps` -> `func hubGreenDeps(prime *lymcwd.Location, subject, rerun string) (greenDeps, error)`
- `cmd/lym#testOnlyCodeAllowed`
- `internal/fronttask#actions`
- `internal/fronttask#TestSettle`
- `internal/adoptcli#TestFixBody`
- `internal/adoptcli#TestRunCheck`
- `internal/adoptcli#TestCheckView`
- `internal/adoptcli#TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers`
- `internal/adoptcli#TestFailedEnd`
- `internal/adoptcli#fixFixture`
- `internal/adoptcli#newFixFixture`
- `internal/adoptcli#fixFixture.tries`
- `internal/adoptcli#fixFixture.setTries`
- `internal/adoptcli#TestGreenEnd`
- `internal/adoptcli#greenFixture`
- `internal/adoptcli_test#TestCheckCmd_FromABeamExitsZeroWhileOneRuns`
- `internal/beamcli#dobbyProgram`

**Uses:**
- `internal/orchcli/doc.go`
- `internal/beamcli/doc.go`
- `internal/fronttask/doc.go`
- `internal/adoptcli#judgeOwnFix`
- `internal/boardengine#FixHold`
- `internal/boardengine#ClearHold`
- `internal/boardengine#HoldRefusal`
- `internal/boardengine#PullBolt`
- `internal/boardengine#WriteHold`
- `internal/boardengine#Registry.ReadsAsHere`
- `internal/fronttask#File`
- `internal/orchcli#NotifyFrom`
- `internal/orchcli#NotifyFromScoped`
- `internal/orchcli#BeamUp`
- `internal/tenderengine#SyncBoltHold`
- `internal/tendercli#HubDeps`
- `internal/pauserecord#BoltHoldCaller`
- `internal/adoptcli#checkFixture`
- `internal/adoptcli#newCheckFixture`

**Intent:**
D6 steps 7 and 8 and `lym adopt check` share one check-and-end, `runHoldCheck`, which reads this machine's hold and keeps the try counts in it.
Every way forward it, the check, the fix loop and the green end word names the re-run over the hold's subjects, `lym adopt <hold's subjects>`, since a one-subject re-run under a multi-subject hold is refused as another set (card 2);
fronttask's hand step, which knows only the subject, names `lym beam status`, which shows that re-run.

`holdCheckDeps` seams: `PullBolt`; `Hold(first)` (the hold standing in the local Bolt, with a flag for whether it reads as this machine's through `ReadsAsHere`, found false when none: this machine's hold whose subjects include `first`, else this machine's first hold, else the other machine's first hold; one machine's holds stand beside each other, while `WriteHold` writes nothing over another machine's); `Check(subject, rerun)` (one `runCheck`, reporting whether it ran); `Fix(subject, tries, record, rerun)` (the fix loop, returning the task filed or standing, the lines, and the `*HoldRefusal` its `FixHold` met, nil when none); `Green(subject, rerun)` (the green end's lines, `rerun` the same re-run `runHoldCheck` builds for `Check` and `Fix`); `ClearHold`; `DropBoltHold`; `BeamUp(subject)`, nil bringing nothing up; `Notice(subject, line)`; `LimitNotice(subject, line)`.
`holdCheckResult` carries `Green`, `Ran`, per-subject `States`, `FixTask`, `HoldLost` (this machine's hold no longer stood at the check's end, so nothing was brought up) and `Lines`.

`runHoldCheck(first)`:
1. Pulls Bolt and reads the `Hold` seam. A pull that fails returns its error naming the re-run once origin answers.
   None: the result names `no_hold` and the line `no hold of this machine stands for <first>, so nothing was checked; way forward: run lym adopt <first>, which writes the hold and checks`; it is no error, so a fix task's landing on a machine whose hold cleared ends quietly.
   A hold of another machine ends the same way, `no_hold` and no error, its line naming that machine as the hold's holder.
   A `first` outside the hold's subjects refuses naming the hold's subjects, way forward `lym adopt <hold's subjects>`.
2. Runs `Check` for every subject of the hold, `first` first, then the rest in hold order. A check that did not run (its lock held) ends the call with `Ran` false and the line `the adopt check of <subject> is already running and posts its own end; way forward: wait for it, then run lym adopt <hold's subjects> if the hold still stands`.
3. Every check green: first pulls Bolt and reads the `Hold` seam once more.
   A pull that fails ends the result not green with its error and the re-run once origin answers, and calls no later seam.
   When no hold of this machine stands, the result ends not green with `HoldLost` set and the line naming the machine whose hold that read found, or saying the hold cleared when it found none, and calls neither `Green`, `ClearHold`, `BeamUp` nor `DropBoltHold`, so a fix task another machine's take-hold now owns stays standing.
   Otherwise `Green` per subject (removes its fix task), then `ClearHold`, which still judges the window left after that read, then `BeamUp` per subject, then `DropBoltHold` last (this machine's bolt-hold entries go at once, not at the dobby's next tick); `DropBoltHold` reads the pause record once more after the drop and drops again any bolt-hold entry a dobby tick that read the hold before the clear wrote meanwhile. The runs go on by the act rule with no hand lift.
   A `ClearHold` that refuses with a `*HoldRefusal` whose `OwnStands` is false (another machine took the hold during the check, or it cleared) or reports false (no hold stood) ends the result not green with `HoldLost` set, no `BeamUp` and no `DropBoltHold`, and the line naming the refusal's machine, or saying the hold cleared meanwhile, so nothing is brought up here.
   A `ClearHold` that refuses with `OwnStands` true (a second process here changed this machine's hold) or fails otherwise ends the result not green with the line naming its error and the re-run `lym adopt <hold's subjects>`, whose check goes green again and clears the hold.
   A failed `BeamUp` adds `runBeamSteps`'s line naming `lym beam start <subject>`; a failed `DropBoltHold` adds a line saying the worktree's dobby lifts the entries at its next tick, or `lym beam start <subject>` when none runs. Neither undoes the clear.
4. Otherwise the first failing subject in hold order gets `Fix` with its `Tries` from the hold; its filing writes the `adopt-fix` hold naming the task with that subject's tries plus one, through `FixHold`, between the entry's creation and its release (`fronttask.File`'s `between`). `BeamUp` per subject follows, so the fix task's dobby starts it.
   A `FixHold` that refuses with a `*HoldRefusal` whose `OwnStands` is false (another machine took the hold during the check, or no hold stands) ends the result with `HoldLost` set and no `BeamUp`, its line carrying the refusal, which names the other machine when one took the hold.
   One with `OwnStands` true (a second process here changed this machine's hold) sets no `HoldLost`: its line carries the refusal and the re-run `lym adopt <hold's subjects>`, and `BeamUp` per subject follows, since the hold is still this machine's.
   Under an open, healthy own fix task, as on a re-run under this machine's own `adopt-fix` hold or after a `--take-hold`, no second task is filed; `judgeOwnFix` reads an own task with no local run record as starting, not halted, which is the state on the machine that took the hold over.
   At the limit nothing is filed, the hold stays, and the line names the failing tests, the fix task and the way forward: settle the failure by hand, then run `lym adopt <hold's subjects>`, whose check removes that task.
5. Every line is posted through `Notice`; the limit line goes through `LimitNotice`, the dobby's notice of the subject's worktree.

`checkDeps` drops `Green` and `Failed` and gains `Rerun`, the hold's re-run, which `runCheck`'s owner-read, owned-elsewhere, reconcile-failure and gate-stopped lines name in place of `lym adopt <subject>`; `runCheck` posts no end lines; it still writes the record, the marks and the tier notices.
`resolveSubjectWorktree`'s error names `lym beam start <subject>` and leaves the re-run to its caller, which `runHoldCheck` appends.
`checkView`'s ended-without-a-verdict line names the re-run `lym beam status` shows under the adopt hold (card 6), not `lym adopt <subject>`.
`hubCheckDeps` takes `terminal`: non-nil, the gate's output goes to the suite log and to it (`io.MultiWriter`), so the foreground adopt shows the gate on the terminal; `checkCmd` passes nil.
`hubCheckDeps` leaves `checkDeps.Rerun` unset: the `Check` closure of `hubHoldCheckDeps` sets it from its `rerun` argument.
`fixDeps` gains `Tries`, `Rerun` and `FixHold(task string, tries int) error`; `hubFixDeps` sets `FixHold` over `boardengine.FixHold`, passing the subject's tries as a one-entry map, and the `Fix` closure of `hubHoldCheckDeps` sets `Tries` and `Rerun` from its `tries` and `rerun` arguments after `hubFixDeps` returns; `failedEnd` reads the count from `Tries` instead of `fixtries.Read`, calls `FixHold` in the filing's `between`, and returns the task filed or the own task standing, and the `*HoldRefusal` `FixHold` returned.
`fixReadError` and `fixFilingError` name the hold standing meanwhile and `Rerun`, with no count reset.
`fixBody`'s stop condition and `fixRemovalError`'s re-run name `Rerun` from `fixDeps`.
`greenEnd` only removes the subject's fix task; `greenDeps` drops `Dir`, whose only use was the count reset, and gains `Rerun`, which `removeFixTask`'s board-read line names; `hubGreenDeps` takes `rerun`, sets `greenDeps.Rerun` from it and no longer sets `Dir`.
fronttask's `actions` row `adopt-check` words its hand step as `lym beam status`, which shows the hold's re-run (card 6).
`hubHoldCheckDeps` fills the seams over the hub: `boardengine.PullBolt`, `ReadOwners` with `machinename.ReadIdentity`, `hubCheckDeps(prime, subject, terminal)` with `runCheck` after setting `Rerun`, `hubFixDeps` with `failedEnd` after setting `Tries` and `Rerun`, `hubGreenDeps(prime, subject, rerun)` with `greenEnd`, `boardengine.ClearHold`, `DropBoltHold` through `tendercli.HubDeps` and `tenderengine.SyncBoltHold` with `Standing` false over the scopes of the live `BoltHoldCaller` entries, `BeamUp` through `orchcli.BeamUp` with `SkipOrch` from `noOrch` (refused under test), `Notice` through `orchcli.NotifyFrom` and `LimitNotice` through `orchcli.NotifyFromScoped` with the subject's scope and `orchcli.DobbyProgram`.
`orchcli.DobbyProgram` is the dobby's program name, which beamcli's `dobbyProgram` now takes its value from, so the two never drift.
`checkCmd` runs `runHoldCheck` over its subject and prints `state`, `fix_task` and `lines`, exiting 0 once the call returns a result (a failed check that filed its next task, a lock-held check and `no_hold` included); an error exits non-zero.
`fronttask`'s `adopt-check` row starts the check detached and reads neither its exit code nor its output.
check.go's file comment says the check posts its tier notices and leaves its end to `runHoldCheck`.
doc.go is rewritten in card 9 with the rest of the flow.
`hubHoldCheckDeps` under `checkCmd` reaches `FixHold`, `ClearHold` and `HoldRefusal.Error` from a main, so `cmd/lym`'s `testOnlyCodeAllowed` drops the `internal/boardengine.FixHold`, `internal/boardengine.ClearHold` and `internal/boardengine.HoldRefusal.Error` entries card 2 added; the exact keys are the ones the guard reports at this card's tree.

`TestRunHoldCheck` (tier 0, over fakes of every seam) rows:
`NoOwnHoldEndsQuietlyNamingTheAdopt` (also a hold of another machine: `no_hold`, the line naming that machine), `APullThatFailsReturnsItsError`, `ASubjectOutsideTheHoldRefusesNamingTheHoldsSubjects`,
`AllGreenRemovesFixTasksClearsTheHoldBringsEachSubjectUpAndDropsBoltHoldEntriesLast`,
`AHoldTakenOverDuringAGreenCheckBringsNothingUp` (`ClearHold` refuses naming the other machine; also a `ClearHold` that reports false; no `BeamUp`, no `DropBoltHold`, `HoldLost` set),
`AHoldTakenOverBeforeTheGreenEndLeavesTheFixTaskStanding` (`Hold` reads another machine's hold after the checks; the fake `Green` records no call, and no `ClearHold`, `BeamUp` or `DropBoltHold`, `HoldLost` set, the line naming that machine),
`AHoldTakenOverDuringAFailingCheckBringsNothingUp` (`Fix` returns `FixHold`'s refusal; no `BeamUp`, `HoldLost` set),
`ARefusalOverThisMachinesChangedHoldIsNotLost` (`ClearHold` refuses with `OwnStands` true: not green, `HoldLost` unset, the line names the re-run, no `BeamUp`; `Fix` returns a refusal with `OwnStands` true: `HoldLost` unset and `BeamUp` per subject),
`APullBeforeTheGreenEndThatFailsCallsNoLaterSeam`,
`AFailedClearHoldNamesTheRerunAndIsNotGreen`, `AFailedBeamUpNamesBeamStart`,
`AFailedSubjectFilesOneTaskWritesAdoptFixWithTriesPlusOneAndBringsEachSubjectUp`,
`TheTryCountIsReadFromTheHold`, `AtTheLimitTheHoldStaysAndTheDobbyNoticeNamesTheFailingTestsTheTaskAndTheRerun`,
`AnOpenOwnFixTaskFilesNoSecondTask`, `ACheckThatDidNotRunEndsNothingAndSaysToWait`, `ANilBeamUpBringsNothingUp`.
`TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry` (tier 1, over a hubforge hub whose pause record holds two `BoltHoldCaller` entries and one operator slug entry) calls the filled `DropBoltHold` and asserts no `bolt-hold` entry is left and the operator's entry stands.
`TestHubHoldCheckDeps_FilledSeamsNameTheRerun` (tier 1, over a hubforge hub) drives the filled seams with a two-subject re-run, so a missed assignment in the hub wiring fails it where the fake-seam rows cannot:
`TheCheckOfASubjectOwnedElsewhereNamesTheRerun` (the subject's owner record names another machine; the filled `Check` reports the owned-elsewhere line naming the re-run, and no gate runs) and
`TheFixAtTheLimitNamesTheRerun` (the filled `Fix` gets a failed record and tries at the subject's limit; no task is filed, and the limit line names the re-run).
`TestRunCheck` (tier 0) drops its end rows, which `TestRunHoldCheck` now holds, asserts no end line is posted, and asserts its owner-read, owned-elsewhere and reconcile-failure lines name `Rerun`.
`TestFixBody` (tier 0) asserts the stop condition names the re-run it is given.
`TestSettle` (tier 0) asserts the failed `adopt-check` action's way forward names `lym beam status`.
`TestCheckView` (tier 0) asserts the new ended-without-a-verdict wording.
`TestFailedEnd` (tier 0) seeds the count through `Tries` (`fixFixture.tries`/`setTries` read and set the fixture's count), sets `Rerun` to a two-subject re-run that its way-forward assertions name, asserts `FixHold` receives the task and tries plus one, asserts the returned task, and gains `AnOwnFixTaskWithNoLocalRunRecordFilesNoSecondTask` and `AFixHoldRefusalComesBackAsLost`.
`TestGreenEnd` (tier 0) drops the count-reset assertion, `greenFixture` drops `Dir` from the deps it builds and its `dir` and `otherDir` fields, which only that assertion read, and `TestGreenEnd` gains `ABoardReadFailureNamesTheRerun` (the fixture's `Tasks` fails; the line names `Rerun`).
`TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` (tier 1) passes a nil terminal.
`TestCheckCmd_FromABeamExitsZeroWhileOneRuns` (tier 1) plants this machine's hold over the beam with `WriteHold`, so the held-lock path is reached, and asserts exit 0 with the check reported already running.

**ImpactSummary:** adoptcli's check, fix loop, green end and check view, one constant moved to orchcli for beamcli's dobby program, fronttask's `adopt-check` hand step, and three test-only-code allowlist entries dropped; `lym adopt check`'s envelope gains `fix_task` and `lines`, and its only caller is fronttask's `adopt-check` row, which starts the check detached and reads neither its exit code nor its output.

**Commit:** 8: Keep the adopt check's try counts in the hold and clear it on green
