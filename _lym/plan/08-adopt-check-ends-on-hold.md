# Card 8 — The adopt check's ends work through the hold

**Create:**
- `plan:internal/adoptcli#holdCheckDeps` -> `type holdCheckDeps struct`
- `plan:internal/adoptcli#holdCheckResult` -> `type holdCheckResult struct`
- `plan:internal/adoptcli#runHoldCheck` -> `func runHoldCheck(ctx context.Context, deps holdCheckDeps, first string) (holdCheckResult, error)`
- `plan:internal/adoptcli#hubHoldCheckDeps` -> `func hubHoldCheckDeps(prime *lymcwd.Location, terminal io.Writer, noOrch bool) (holdCheckDeps, error)`
- `plan:internal/adoptcli#TestRunHoldCheck` -> `func TestRunHoldCheck(t *testing.T)`
- `plan:internal/adoptcli#TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry` -> `func TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry(t *testing.T)`
- `plan:internal/orchcli#DobbyProgram` -> `const DobbyProgram = "DOBBY"`

**Edit:**
- `internal/adoptcli#checkDeps`
- `internal/adoptcli#runCheck`
- `internal/adoptcli#hubCheckDeps` -> `func hubCheckDeps(prime *lymcwd.Location, subject string, terminal io.Writer) (checkDeps, error)`
- `internal/adoptcli#resolveSubjectWorktree`
- `internal/adoptcli#checkCmd`
- `internal/adoptcli#checkView`
- `internal/adoptcli#fixDeps`
- `internal/adoptcli#failedEnd` -> `func failedEnd(deps fixDeps, record CheckRecord) (fixTask string, lines []string)`
- `internal/adoptcli#hubFixDeps`
- `internal/adoptcli#fixReadError`
- `internal/adoptcli#fixFilingError`
- `internal/adoptcli#greenDeps`
- `internal/adoptcli#greenEnd`
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
- `internal/adoptcli#judgeOwnFix`
- `plan:internal/boardengine#FixHold`
- `plan:internal/boardengine#ClearHold`
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
Every way forward it words names the re-run over the hold's subjects, `lym adopt <hold's subjects>`, since a one-subject re-run under a multi-subject hold is refused as another set (card 2).

`holdCheckDeps` seams: `PullBolt`; `Hold` (this machine's hold in the local Bolt, found false when none, compared through `ReadsAsHere`); `Check(subject, rerun)` (one `runCheck`, reporting whether it ran); `Fix(subject, tries, record, rerun)` (the fix loop, returning the task filed or standing and the lines); `Green(subject)` (the green end's lines); `ClearHold`; `DropBoltHold`; `BeamUp(subject)`, nil bringing nothing up; `Notice(subject, line)`; `LimitNotice(subject, line)`.
`holdCheckResult` carries `Green`, `Ran`, per-subject `States`, `FixTask` and `Lines`.

`runHoldCheck(first)`:
1. Pulls Bolt and reads this machine's hold. A pull that fails returns its error naming the re-run once origin answers.
   None: the result names `no_hold` and the line `no hold of this machine stands for <first>, so nothing was checked; way forward: run lym adopt <first>, which writes the hold and checks`; it is no error, so a fix task's landing on a machine whose hold cleared ends quietly.
   A `first` outside the hold's subjects refuses naming the hold's subjects, way forward `lym adopt <hold's subjects>`.
2. Runs `Check` for every subject of the hold, `first` first, then the rest in hold order. A check that did not run (its lock held) ends the call with `Ran` false and the line `the adopt check of <subject> is already running and posts its own end; way forward: wait for it, then run lym adopt <hold's subjects> if the hold still stands`.
3. Every check green: `Green` per subject (removes its fix task), then `ClearHold`, then `BeamUp` per subject, then `DropBoltHold` last (this machine's bolt-hold entries go at once, not at the dobby's next tick); `DropBoltHold` reads the pause record once more after the drop and drops again any bolt-hold entry a dobby tick that read the hold before the clear wrote meanwhile. The runs go on by the act rule with no hand lift.
   A failed `ClearHold` ends the result not green with the line naming its error and the re-run `lym adopt <hold's subjects>`, whose check goes green again and clears the hold.
   A failed `BeamUp` adds `runBeamSteps`'s line naming `lym beam start <subject>`; a failed `DropBoltHold` adds a line saying the worktree's dobby lifts the entries at its next tick, or `lym beam start <subject>` when none runs. Neither undoes the clear.
4. Otherwise the first failing subject in hold order gets `Fix` with its `Tries` from the hold; its filing writes the `adopt-fix` hold naming the task with that subject's tries plus one, through `FixHold`, between the entry's creation and its release (`fronttask.File`'s `between`). `BeamUp` per subject follows, so the fix task's dobby starts it.
   Under an open, healthy own fix task, as on a re-run under this machine's own `adopt-fix` hold or after a `--take-hold`, no second task is filed; `judgeOwnFix` reads an own task with no local run record as starting, not halted, which is the state on the machine that took the hold over.
   At the limit nothing is filed, the hold stays, and the line names the failing tests, the fix task and the way forward: settle the failure by hand, then run `lym adopt <hold's subjects>`, whose check removes that task.
5. Every line is posted through `Notice`; the limit line goes through `LimitNotice`, the dobby's notice of the subject's worktree.

`checkDeps` drops `Green` and `Failed` and gains `Rerun`, the hold's re-run, which `runCheck`'s reconcile-failure and gate-stopped lines name in place of `lym adopt <subject>`; `runCheck` posts no end lines; it still writes the record, the marks and the tier notices.
`resolveSubjectWorktree`'s error names `lym beam start <subject>` and leaves the re-run to its caller, which `runHoldCheck` appends.
`checkView`'s ended-without-a-verdict line names the re-run `lym beam status` shows under the adopt hold (card 6), not `lym adopt <subject>`.
`hubCheckDeps` takes `terminal`: non-nil, the gate's output goes to the suite log and to it (`io.MultiWriter`), so the foreground adopt shows the gate on the terminal; `checkCmd` passes nil.
`fixDeps` gains `Tries`, `Rerun` and `FixHold(task string, tries int) error`; `failedEnd` reads the count from `Tries` instead of `fixtries.Read`, calls `FixHold` in the filing's `between`, and returns the task filed or the own task standing.
`fixReadError` and `fixFilingError` name the hold standing meanwhile and `Rerun`, with no count reset.
`greenEnd` only removes the subject's fix task; `greenDeps` drops `Dir`'s use for the count.
`hubHoldCheckDeps` fills the seams over the hub: `boardengine.PullBolt`, `ReadOwners` with `machinename.ReadIdentity`, `hubCheckDeps(prime, subject, terminal)` with `runCheck`, `hubFixDeps` with `FixHold` over `boardengine.FixHold`, `hubGreenDeps` with `greenEnd`, `boardengine.ClearHold`, `DropBoltHold` through `tendercli.HubDeps` and `tenderengine.SyncBoltHold` with `Standing` false over the scopes of the live `BoltHoldCaller` entries, `BeamUp` through `orchcli.BeamUp` with `SkipOrch` from `noOrch` (refused under test), `Notice` through `orchcli.NotifyFrom` and `LimitNotice` through `orchcli.NotifyFromScoped` with the subject's scope and `orchcli.DobbyProgram`.
`orchcli.DobbyProgram` is the dobby's program name, which beamcli's `dobbyProgram` now takes its value from, so the two never drift.
`checkCmd` runs `runHoldCheck` over its subject and prints `state`, `fix_task` and `lines`, exiting 0 once the call returns a result (a failed check that filed its next task, a lock-held check and `no_hold` included), so `fronttask`'s `adopt-check` row does not retry it at every settle; an error exits non-zero.
check.go's file comment says the check posts its tier notices and leaves its end to `runHoldCheck`.
doc.go is rewritten in card 9 with the rest of the flow.

`TestRunHoldCheck` (tier 0, over fakes of every seam) rows:
`NoOwnHoldEndsQuietlyNamingTheAdopt`, `APullThatFailsReturnsItsError`, `ASubjectOutsideTheHoldRefusesNamingTheHoldsSubjects`,
`AllGreenRemovesFixTasksClearsTheHoldBringsEachSubjectUpAndDropsBoltHoldEntriesLast`,
`AFailedClearHoldNamesTheRerunAndIsNotGreen`, `AFailedBeamUpNamesBeamStart`,
`AFailedSubjectFilesOneTaskWritesAdoptFixWithTriesPlusOneAndBringsEachSubjectUp`,
`TheTryCountIsReadFromTheHold`, `AtTheLimitTheHoldStaysAndTheDobbyNoticeNamesTheFailingTestsTheTaskAndTheRerun`,
`AnOpenOwnFixTaskFilesNoSecondTask`, `ACheckThatDidNotRunEndsNothingAndSaysToWait`, `ANilBeamUpBringsNothingUp`.
`TestHubHoldCheckDeps_DropBoltHoldLeavesNoBoltHoldEntry` (tier 1, over a hubforge hub whose pause record holds two `BoltHoldCaller` entries and one operator slug entry) calls the filled `DropBoltHold` and asserts no `bolt-hold` entry is left and the operator's entry stands.
`TestRunCheck` (tier 0) drops its end rows, which `TestRunHoldCheck` now holds, asserts no end line is posted, and asserts its reconcile-failure line names `Rerun`.
`TestCheckView` (tier 0) asserts the new ended-without-a-verdict wording.
`TestFailedEnd` (tier 0) seeds the count through `Tries` (`fixFixture.tries`/`setTries` read and set the fixture's count), asserts `FixHold` receives the task and tries plus one, asserts the returned task, and gains `AnOwnFixTaskWithNoLocalRunRecordFilesNoSecondTask`.
`TestGreenEnd` (tier 0) drops the count-reset assertion.
`TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` (tier 1) passes a nil terminal.
`TestCheckCmd_FromABeamExitsZeroWhileOneRuns` (tier 1) plants this machine's hold over the beam with `WriteHold`, so the held-lock path is reached, and asserts exit 0 with the check reported already running.

**ImpactSummary:** adoptcli's check, fix loop, green end and check view, one constant moved to orchcli for beamcli's dobby program; `lym adopt check`'s envelope gains `fix_task` and `lines`, and its only caller is fronttask's `adopt-check` row, which reads the exit code alone.

**Commit:** 8: Keep the adopt check's try counts in the hold and clear it on green
