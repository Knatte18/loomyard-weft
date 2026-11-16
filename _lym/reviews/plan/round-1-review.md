---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/11-cancelled-record-removal.md (cleanupDeps.PairStands, judgeRunDirectory); _lym/plan/04-runfsm-act-rule.md (Verdict.WayForward)
    summary: Cleanup judges whether a kept cancel's pair stands from local pair parts only, so on a machine without the pair it deletes, commits and pushes a kept cancel's batten record, and card 4's way forward sends the operator there; D1 and D3 say the record reads cancelled on every machine and cleanup refuses it while its pair stands.
    fix: "In card 11, fill `PairStands` from `Topology.PairParts` or `Topology.TaskBranchOnOrigin`: the pair stands when any local part is present or the task branch is on origin, the input D1 already uses. Only a cancelled record with no local part and no task branch on origin is a removal candidate. Add a `TestCleanup` row for a kept cancel whose pair is absent here but whose task branch is on origin, skipped with the way forward. In card 4, make `WayForward` for every `cancelled` reading with `PairKept` name `lym fabric remove <slug>`, run on the machine that holds the pair when `PairPresent` is false, and drop the `lym batten cleanup --apply` way forward."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/14-tender-pause-intent.md (tenderengine#Pause, tendercli#HubDeps)
    summary: Card 14 puts the owner refusal and the intent write in tenderengine.Pause for every slug pause, but tendercli.PauseOwnedElsewhere also calls Pause with a slug, under the OwnerCaller and through HubDeps. Its purpose is a run another machine owns, so the owner pause that batten's PauseOwned seam and card 6's hand-over handling rely on is refused.
    fix: "In card 14, gate the owner refusal and the intent write on the operator's own slug pause, never on a pause whose caller is `pauserecord.OwnerCaller`. One way: `Pause` skips `RunOwner` and `PauseIntent` when `req.Caller` is `pauserecord.OwnerCaller`. The other: move both calls out of `tenderengine.Pause` into `tendercli#pauseCmd`'s slug path. Add a `TestPause` row where an OwnerCaller slug pause of a run owned elsewhere writes its entry and no intent. `TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller` must keep passing."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/09-cancel-intent-in-batten-record.md (Edit list)
    summary: Card 9 retypes or renames the marker seams but does not list the tests and fixtures that call them. Card 10 then deletes cancelmarker.go while TestArmAt_StartGuardRefusesBeforeAnyWrite still calls takeCancelMarker, so battencli's test binary breaks. TestCancelIntegration_BlockedRunWithNoPair still asserts records_removed true for a kept cancel.
    fix: "Add to card 9's **Edit:** `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`, `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`, `internal/battencli#cancelFake`, `internal/battencli#cancelFake.deps`, `internal/battencli#newPreDriveReceiver` and `internal/pairteardown#fakeSubstrates.teardown`. Say in the Intent that their marker fixtures become an intent in the record. Say that `TestCancelIntegration_BlockedRunWithNoPair` asserts `records_removed` false and `record_kept` true for its kept cancel. In card 10's Intent, drop the claim that `TestCancelIntegration_BlockedRunWithNoPair` loses only its marker check."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/09-cancel-intent-in-batten-record.md (Edit and Extends lists)
    summary: Card 9 falsifies three texts it does not list for rewrite. The battencli package doc in cli.go describes the old cancel order. The cancel verb's Long help says it removes the records and leaves its marker. pairteardown/doc.go is listed only under Extends, though its sentence about the cancel marker must be rewritten.
    fix: "In card 9, add `internal/battencli/cli.go` and `internal/battencli#battenCLI.cancelCmd` under **Edit:**, and move `internal/pairteardown/doc.go` from **Extends:** to **Edit:**. The Intent states three rewrites: the package doc's cancel order (intent recorded and pushed first, the record kept with its finish without --remove, deleted with the pair under --remove, no marker); the `preDrive` field comment; and `cancelCmd`'s Long help and output key list, `intent_pushed` and `record_kept` included."
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/02-shed-cancelled-state-intents.md (ImpactSummary); _lym/plan/09-cancel-intent-in-batten-record.md (Edit list)
    summary: "The new cancelled state reaches battencli's exhaustive state switches, which no card lists: battenCLI.checkRunStatus (arm.go:530), battenCLI.battenPreStepUnmarked (arm.go:902) and battenRunEnvelope (drive.go:164). A kept cancel's record then gets a bare unrecognized-state error with no way forward."
    fix: "Add `internal/battencli#battenCLI.checkRunStatus`, `internal/battencli#battenCLI.battenPreStepUnmarked` and `internal/battencli#battenRunEnvelope` to card 9's **Edit:**. Give each a `cancelled` branch that refuses, naming `lym fabric remove <slug>` for a kept cancel. In `battenRunEnvelope`, classify a status that a cancel finished during a pass as an ended run. Add the matching rows to `TestStartGuard_Refuse` or `TestPreDriveCheck`. Fix card 2's ImpactSummary so it no longer says only step, goto and relaunch read the state."
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/13-fixer-attempts-tracked.md (Edit list, Tests paragraph)
    summary: Card 13 moves the attempt records to AttemptsDir and lists only the fixture constructors. The fixture methods and tests still build attempt paths from fx.deps.ScratchDir, so they seed and read the wrong directory, and the card's claim that these tests keep passing through their fixtures is false.
    fix: "Add to card 13's **Edit:** `internal/landingshed#mergeInFixture.seedAttempts`, `internal/landingshed#mergeInFixture.attemptsOnDisk`, `internal/landingshed#finalizeGateFixture.seed`, `internal/landingshed#finalizeGateFixture.attempts`, `internal/landingshed#TestMergeIn_Fixer`, `internal/landingshed#TestFixerRun_Outcomes` and `internal/landingshed#TestFinalize_GateAndFixer`. Each builds its attempts path from `deps.AttemptsDir`. Replace the sentence that the existing tests keep passing through their fixtures with that statement."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/11-cancelled-record-removal.md (Teardown.Run seam)
    summary: The record deletion runs after every successful Teardown.Run removal. lym fabric remove sends Request.Local for an owned_elsewhere or ended verdict, so a local-only removal on a machine that does not own the run deletes, commits and pushes the batten record while the pair stands elsewhere.
    fix: "In card 11, run `deleteCancelledBattenRecord` from `Teardown.Run` only when `req.Local` is false. Add a `TestTeardown_CallSequence` row where a local removal never calls the seam."
  - id: F8
    severity: MEDIUM
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/09-cancel-intent-in-batten-record.md (cancel order)
    summary: Card 9 does not say what the cancel does when RecordCancelIntent reports no batten record. refuseNothingLeft lets the cancel go on with only a pair part, a session, a claim or processes left, so it would end processes with no intent recorded anywhere, against D3's intent pushed before anything is ended.
    fix: "In card 9's cancel order, state the no-record branch. Either proceed as today with no intent and a stated bound: nothing of the run is on origin for another machine to read, so it is recorded only in the cancel's report. Or refuse naming a way forward. Add the matching `TestCancelRun` row."
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/03-runfsm-states-derive.md (Uses battenrecipe); _lym/plan/05-runfsm-tracked-reads.md (Read via hubgeom.ParentOf); _lym/plan/14-tender-pause-intent.md (FetchOwner via hubgeom.ParentOf)
    summary: D1 names runfsm a leaf package that every reader of a run's state takes the state from, but the plan imports packages that reach dobbyengine and landingshed. battenrecipe imports shedbuild and hubgeom imports dobbyengine, both reaching those packages, so dobbyengine and landingshed could never import runfsm without a cycle.
    fix: "Keep runfsm's imports free of every package that reaches dobbyengine or landingshed. In card 3, drop the `internal/battenrecipe#RowsBeforeRunShed` Uses. Add a `BattenBeforeRunShed` field to `Records`, told by the reader, and state the import bound in `runfsm/doc.go`. In card 5, `Read` takes `parentOf` and the batten rows before Run-Shed from its caller, as `OwnerOf` already takes `parentOf`. In card 14, `FetchOwner` takes `parentOf` the same way. The callers in cards 6, 7, 8 and 14 pass `hubgeom.ParentOf` and `battenrecipe.RowsBeforeRunShed()`."
  - id: F10
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/12-pull-request-record-tracked.md (Intent)
    summary: "D3 says a session id whose transcript is absent here starts that session fresh. Card 12 makes the PR record, SessionID included, travel between machines but does not carry that. PRReview.ensureSession passes rec.SessionID as ResumeSessionID, and claudeengine's resume check refuses a session with no transcript here, so PR-Review halts on the new machine."
    fix: "Add `internal/landingshed#PRReview.ensureSession` to card 12's **Edit:**. When the recorded session cannot be resumed here because its transcript is absent, start a fresh session and record the new ids. Detect this through shuttle's resume capability, keeping the transcript check in claudeengine per PATTERN-shuttle-provider-seam. Add a tier0 row with a sessions fake whose resume check reports no transcript, asserting a fresh start."
  - id: F11
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/03-runfsm-states-derive.md (Derive landing condition)
    summary: "Card 3 reads landing for a child whose history holds a Finalize entry, but D1 says the child run at Finalize or a later row. Goto admits moving a run back from Finalize to an earlier row, so a re-opened child at Webster keeps its Finalize entry and misreads as landing."
    fix: "In card 3, take `landing` from the child's current row alone: Finalize, or a row the loom recipe lists after Finalize. Name the row source in the card. Add a `TestDerive` row where a child sent back from Finalize to an earlier row reads `running`."
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/02-shed-cancelled-state-intents.md (Create TestCancelledRun_NeverStepsGoesToOrRelaunches)
    summary: The new test re-covers goto and relaunch refusals that existing tables already cover. TestGoto_RefusalNamesWayForward and TestRelaunch each hold a done-run refusal row with an unchanged-file check, so a cancelled status is one more row in each.
    fix: "In card 2, replace the Create of `TestCancelledRun_NeverStepsGoesToOrRelaunches` with `plan:internal/shedengine#TestStep_RefusesACancelledRun`, which covers the step refusal no table holds. Add `internal/shedengine#TestGoto_RefusalNamesWayForward` and `internal/shedengine#TestRelaunch` under **Edit:**, each gaining a cancelled row beside its done row."
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/08-loom-resume-asks-act-rule.md (Create TestResumeVerb_RefusesWhatTheActRuleWaitsOrHandsOver)
    summary: The new test function repeats the shape of TestResumeVerb. That table is already built on newResumeVerbReceiver and holds the covered refusal row checking the way forward, so the act-rule refusals are rows of it.
    fix: "In card 8, drop the Create of `TestResumeVerb_RefusesWhatTheActRuleWaitsOrHandsOver` and add `internal/loomcli#TestResumeVerb` under **Edit:**. It gains a fake `actOf` column: a `wait` row and a `hand_over` row refuse with the way forward and leave the park marker, and the existing rows run with an `act` verdict."
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/14-tender-pause-intent.md (Create TestResume_RefusesWhatTheActRuleWaitsOrHandsOver)
    summary: The new tier1 test builds a second hub to drive one slug resume refusal. TestPauseAndResume_WriteLiftAndRefuseOverTheHub, which card 14 already edits, drives slug pause and resume over the same kind of hub and asserts its refusals.
    fix: "In card 14, drop the Create of `TestResume_RefusesWhatTheActRuleWaitsOrHandsOver`. Add its case to `TestPauseAndResume_WriteLiftAndRefuseOverTheHub`: a slug whose record carries a cancel intent is refused by `resume` with its way forward and keeps its entry."
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/09-cancel-intent-in-batten-record.md (Edit internal/battencli#refuseNothingLeft)
    summary: Card 9 replaces cancelMarkerReading with cancelReading everywhere, which changes refuseNothingLeft's reading parameter type, but its Edit glyph has no re-sign arrow.
    fix: "In card 9, write the bullet as `internal/battencli#refuseNothingLeft` -> `func refuseNothingLeft(slug string, remove, endOnly bool, rerun string, reading cancelReading, parts fabricengine.PairParts, deps cancelDeps) error`."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/11-cancelled-record-removal.md (Edit list)
    summary: judgeStatus's skip reason for a run that is not done says lym batten cancel ends it and a later cleanup removes it. After cards 9 and 11, a cancel without --remove keeps the record and cleanup refuses it while its pair stands, so the clause misleads.
    fix: "Add `internal/battencli#judgeStatus` to card 11's **Edit:**. Reword the skip reason: `lym batten cancel <slug> --remove` ends the run and deletes the record with its pair, or a kept cancel's record goes with `lym fabric remove <slug>`."
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/11-cancelled-record-removal.md (Teardown.Run follow-up)
    summary: BattenFollowUp gives a record in any state but done the advice to run lym batten cancel --remove. A cancelled record left after a failed record deletion in Teardown.Run is therefore told to re-cancel a finished cancel instead of to retry the removal.
    fix: "Add `internal/pairteardown#BattenFollowUp` to card 11's **Edit:**. Give a `cancelled` record the follow-up `lym fabric remove <slug>` again, or `lym batten cleanup --apply` once the pair is gone. State in card 11 that the seam runs before `Teardown.Run` reads the follow-up."
  - id: F18
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/12-pull-request-record-tracked.md (Edit list)
    summary: Card 12 moves the PR record out of scratch but does not list two members that describe it there. One is TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord, whose name and assertions are about the record in scratch. The other is PullRequestRecordLock, whose doc says the lock sits beside the record.
    fix: "Add `internal/loomcli#TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord` and `internal/shedrun#PullRequestRecordLock` to card 12's **Edit:**. Update the test's assertions, and its name to match, for the record in the run's records. Change the lock's doc comment to say it stays under ScratchDir while the record sits under RunDir."
  - id: F19
    severity: LOW
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/07-batten-run-asks-act-rule.md (startGuard Act seam); _lym/plan/14-tender-pause-intent.md (Deps.RunOwner, Deps.PauseIntent)
    summary: Cards 7 and 14 add function seams without saying what a nil seam does. Card 8 says a nil actOf reads as act, and every other tenderengine.Deps func field documents its nil reading, so hand-built receivers and Deps literals in unlisted tests would call a nil func.
    fix: "In card 7, state that a nil `startGuard.Act` reads as `act`. In card 14, state that a nil `RunOwner` reads as owned here and online, and that a nil `PauseIntent` records none. Each field's doc comment says so."
  - id: F20
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-cancel-intent-in-batten-record.md (startGuard rows)
    summary: Card 7 asks the act rule first in startGuard.refuse, and a cancel intent without a finish derives cancelling and answers wait. Card 9's intent refusal can therefore never be reached for the verb run, only for step and goto.
    fix: "In card 9, keep the start guard's intent refusal rows in `TestStartGuard_Refuse` to `step` and `goto`. For `run`, say that card 7's act-rule refusal answers first and the `run` rows assert its text."
  - id: F21
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/14-tender-pause-intent.md (resumeCmd act check)
    summary: The act check on lym tender resume of a slug answers hand_over for a run another machine owns. It therefore refuses the hand lift of that slug's owner pause on this machine, which pauserecord's doc names as one of its two lifts, and no card updates that contract.
    fix: "In card 14, state that `lym tender resume <slug>` refuses a run owned elsewhere with the way forward `lym adopt <subject>` here first or `lym batten cancel <slug>`. Add `internal/pauserecord/doc.go` under **Edit:** to reword the owner-pause lift sentence to match."
  - id: F22
    severity: LOW
    class: decision
    origin: handler
    location: _lym/plan/04-runfsm-act-rule.md (Aside.CancelIntent, Aside.Hold); _lym/plan/00-overview.md
    summary: D1 says every verb or loop that acts on a run asks the act rule, and names lym batten cancel's own asides. The plan has only the four verbs of the Delivers line ask it, so the CancelIntent and Hold asides card 4 builds have no caller in this part, and the plan never states that cancel, batten step and loom start ask in a later part.
    fix: In 00-overview.md, add a Shared Decision naming the verbs this part makes ask the act rule, the four of the Delivers line, and that `lym batten cancel`, `lym batten step` and `lym loom start` ask in a later part. Card 4's Intent says the `CancelIntent` and `Hold` asides wait for that part's callers.
  - id: F23
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/01-lym-files-cache-pattern.md (Intent, reworded bullet)
    summary: The reworded PATTERN-lym-files bullet says a lost sent-notice memory can repeat a notice once, while D3 says it can repeat a notice or a revive once.
    fix: In card 1, make the bullet say a sent-notice memory lost with `.lym` can repeat a notice or a revive once on the new machine.
---

## Findings

### [BLOCKING] Cleanup deletes a kept cancel's record on a machine without its pair

**Location:** `_lym/plan/11-cancelled-record-removal.md`, `_lym/plan/04-runfsm-act-rule.md`

**Issue:** `Topology.PairParts` reads local parts only (`internal/fabricengine/pairparts.go:16,35`). The prime's batten record is tracked and pushed, so on a machine that never held the kept pair, card 11 judges the record a removal candidate. Card 4 also names `lym batten cleanup --apply` as the way forward for a cancelled run whose pair is absent here. D3 refuses a cancelled record whose pair stands. D1 and its acceptance line say a kept cancel's record reads `cancelled` on every machine.

### [BLOCKING] Card 14's owner refusal refuses the internal owner pause

**Location:** `_lym/plan/14-tender-pause-intent.md`

**Issue:** `tendercli.PauseOwnedElsewhere` builds its deps through `HubDeps` and calls `tenderengine.Pause` with a slug and `Caller: pauserecord.OwnerCaller` (`internal/tendercli/deps.go:233-238`). Card 14 makes `Pause` with a slug ask `RunOwner` first and refuse a run another machine owns, exempting only a hub-wide pause. The owner pause exists for exactly that run, so it is refused. Card 6's hand-over handling and batten's `PauseOwned` seam both rely on it.

### [MEDIUM] Card 9 misses the tests and fixtures of the seams it changes

**Location:** `_lym/plan/09-cancel-intent-in-batten-record.md`

**Issue:** The unlisted members:
- `TestArmAt_StartGuardRefusesBeforeAnyWrite` calls `takeCancelMarker` (`internal/battencli/startguard_integration_test.go:103-118`), which card 10 deletes.
- `cancelFake.deps` sets `ReadMarker` (`cancel_test.go:141`).
- `newPreDriveReceiver` sets `startGuard.Marker` (`predrive_test.go:29`).
- `fakeSubstrates.teardown` sets the seam card 9 renames, `cancelMarked` (`internal/pairteardown/teardown_test.go:117`).
- `TestCancelIntegration_BlockedRunWithNoPair` asserts `records_removed` true for a cancel without `--remove` (`cancel_integration_test.go:120,162`).

### [MEDIUM] Card 9 falsifies the package doc, the verb help and pairteardown's doc without listing them

**Location:** `_lym/plan/09-cancel-intent-in-batten-record.md`

**Issue:** The cancel order at `internal/battencli/cli.go:19-29` says the cancel takes the marker, removes the records and removes the marker last. The `Long` help at `internal/battencli/cancelcmd.go:54-76` says it removes the records and scratch and leaves its marker offline. `internal/pairteardown/doc.go:154` names the cancel marker, and card 9 lists that file only under Extends.

### [MEDIUM] The cancelled state reaches exhaustive switches no card lists

**Location:** `_lym/plan/02-shed-cancelled-state-intents.md`, `_lym/plan/09-cancel-intent-in-batten-record.md`

**Issue:** Two methods fall to `unrecognized status state` for any state outside their list: `battenCLI.checkRunStatus` (`internal/battencli/arm.go:530-539`) and `battenCLI.battenPreStepUnmarked` (`arm.go:902-908`). `battenRunEnvelope` (`internal/battencli/drive.go:164-170`) does the same at a driver's end.

### [MEDIUM] Card 13's fixture methods still build attempt paths from the scratch directory

**Location:** `_lym/plan/13-fixer-attempts-tracked.md`

**Issue:** Each of these still builds its attempts path with `FixerAttemptsPath(fx...ScratchDir, ...)`:
- `internal/landingshed/parentalign_fixer_test.go:51,59,544,558`
- `internal/landingshed/finalize_gate_test.go:142,150,670`
- `internal/landingshed/fixerrun_test.go:340`

### [MEDIUM] Card 11's deletion runs on local-only removals

**Location:** `_lym/plan/11-cancelled-record-removal.md`

**Issue:** `internal/fabriccli/remove.go:86-104` calls `Teardown.Run` with `Local: true` for an `ended` or `owned_elsewhere` verdict. Card 11's seam has no `Local` check.

### [MEDIUM] Card 9 leaves the cancel of a slug with no batten record unstated

**Location:** `_lym/plan/09-cancel-intent-in-batten-record.md`

**Issue:** `RecordCancelIntent` reports `found` false and writes nothing, while `refuseNothingLeft` (`internal/battencli/cancel.go:675-678`) still lets the cancel end processes on other leftovers.

### [MEDIUM] runfsm is not the leaf D1 names

**Location:** `_lym/plan/03-runfsm-states-derive.md`, `_lym/plan/05-runfsm-tracked-reads.md`, `_lym/plan/14-tender-pause-intent.md`

**Issue:** `go list` shows that `internal/hubgeom` imports `internal/dobbyengine` directly and reaches `internal/landingshed`. `internal/battenrecipe` imports `shedbuild`, which reaches `dobbyengine`, `landingshed` and `hubgeom`. Cards 3, 5 and 14 make runfsm import both, so neither dobbyengine nor landingshed can ever take its run state from runfsm, as D1 requires of every reader.

### [MEDIUM] The tracked PR record's session id is never started fresh

**Location:** `_lym/plan/12-pull-request-record-tracked.md`

**Issue:** `PRReview.ensureSession` sets `spec.ResumeSessionID = rec.SessionID` (`internal/landingshed/prsession.go:80`). `Runner.checkResume` fails the start on the engine's error (`internal/shuttleengine/run.go:969-972`), and `checkResumable` refuses a session with no transcript here (`internal/shuttleengine/claudeengine/resume.go:53-56`).

### [MEDIUM] Card 3 reads landing from a Finalize entry in the history

**Location:** `_lym/plan/03-runfsm-states-derive.md`

**Issue:** `TestGoto_AwaitingRefusesFinalizeAndAdmitsEarlierRow` (`internal/shedengine/goto_test.go:475`) pins that goto moves a run back from Finalize to an earlier row. The run's history keeps its Finalize entry after that move. The tapestry recipe lists Friction-Reflect after Finalize, so D1's "later row" has a recipe answer.

### [MEDIUM] Card 2 adds a test the goto and relaunch tables already cover

**Location:** `_lym/plan/02-shed-cancelled-state-intents.md`

**Issue:** Each table holds a done-run row with the same refusal check:
- `TestGoto_RefusalNamesWayForward` (`internal/shedengine/goto_test.go:151-163`): "done run names seeding a new run", also asserting the file is unchanged.
- `TestRelaunch` (`goto_test.go:583-600`): "done run".

### [MEDIUM] Card 8 adds a test function beside the table that covers its shape

**Location:** `_lym/plan/08-loom-resume-asks-act-rule.md`

**Issue:** `TestResumeVerb` (`internal/loomcli/resume_test.go:155`) is built on `newResumeVerbReceiver`. It already holds the covered refusal row "a parked driver under a pause is refused and not woken" (`resume_test.go:228`).

### [LOW] Card 14's new tier1 test belongs in the hub test it already edits

**Location:** `_lym/plan/14-tender-pause-intent.md`

**Issue:** `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (`internal/tendercli/cli_test.go:116`) drives slug pause and resume over a hub, and card 14 already edits it.

### [LOW] refuseNothingLeft lacks its re-sign arrow

**Location:** `_lym/plan/09-cancel-intent-in-batten-record.md`

**Issue:** Its current reading parameter is `cancelMarkerReading` (`internal/battencli/cancel.go:657`), and card 9 replaces that type with `cancelReading`.

### [LOW] judgeStatus's skip reason goes stale

**Location:** `_lym/plan/11-cancelled-record-removal.md`

**Issue:** The skip reason at `internal/battencli/cleanup.go:549-551` says the cancel ends the run and a later cleanup removes it.

### [LOW] BattenFollowUp advises re-cancelling a finished cancel

**Location:** `_lym/plan/11-cancelled-record-removal.md`

**Issue:** `internal/pairteardown/followup.go:33-40` falls through to `lym batten cancel <slug> --remove` for every state but done.

### [LOW] Card 12 misses two members that place the PR record in scratch

**Location:** `_lym/plan/12-pull-request-record-tracked.md`

**Issue:** `internal/loomcli/pawlwiring_integration_test.go:77` and `internal/shedrun/paths.go:235-239`.

### [LOW] New seams with no stated nil reading

**Location:** `_lym/plan/07-batten-run-asks-act-rule.md`, `_lym/plan/14-tender-pause-intent.md`

**Issue:** Every other func field of `internal/tenderengine/report.go:79-122` documents its nil reading.

### [LOW] Card 9's run row for the intent refusal is unreachable

**Location:** `_lym/plan/09-cancel-intent-in-batten-record.md`

**Issue:** Card 7 asks the act rule before every other check in `startGuard.refuse`.

### [LOW] tender resume's hand_over contradicts the owner-pause lift contract

**Location:** `_lym/plan/14-tender-pause-intent.md`

**Issue:** pauserecord's doc names `lym tender resume` as one of the two hand lifts of an owner pause.

### [LOW] The plan does not say which act-rule callers wait for a later part

**Location:** `_lym/plan/04-runfsm-act-rule.md`, `_lym/plan/00-overview.md`

**Issue:** D1 names `lym batten cancel`, `lym batten step` and `lym loom start` among the verbs that ask the act rule. The Delivers line lists only four verbs.

### [NIT] Card 1's bullet drops the revive

**Location:** `_lym/plan/01-lym-files-cache-pattern.md`

**Issue:** D3 says a lost sent-notice memory can repeat a notice or a revive once.

## Focus verdicts

- Card order: no card uses a symbol a later card creates. Cards 6, 7, 8 and 14 come after cards 3–5, and cards 9, 11 and 14 come after card 2.
- D3 fidelity:
  - The cancel finish, the kept record and the `--remove` deletion are carried by card 9.
  - `lym fabric remove` is carried by card 11.
  - The dobby's existing ended teardown reaches `Teardown.Run` through `ReconcileOrigin`'s `teardownEnded` (`internal/pairteardown/originseams.go:179`), so card 11's seam covers it today.
  - The repair-and-refuse rule is carried by card 11, but with F1's defect.
  - The uncarried D3 step is the fresh start of an absent session (F10).
- D1 act rule: card 4 carries every branch in the record's order, and the shed-pause aside. The Delivers line licenses the zero-valued `Held`, `PushOwedOffline` and `CodeTipMissing`.
- Scope licence:
  - Card 1 is licensed by the declared PATTERN change.
  - Card 10 is licensed by D3's replacement of the cancel marker.
  - No card touches the origin-owed marker.
- Reachability: every prescribed refusal has a named input and test (see the guard-reach lens report). The exceptions are F2, F20 and F21.
- Attack surface:
  - The shed-pause aside on `lym batten run` and `lym loom resume` keeps today's behaviour, since both already resume a paused run.
  - Card 6 bounds the dobby's aside.
  - Card 14 bounds the pause-coverage aside by the slug's own entries.
  - No finding beyond F19.
- Verify lines: no card carries a `**Verify:**` line.
- Research steps and the board entry: no card leaves a fact to be found while it runs, edits this task's board entry, or files or promotes a task.
- PATTERN fit:
  - PATTERN-fabric-boundary and PATTERN-gitrepo-client-boundary hold: runfsm and pairteardown reach git only through `fabricengine`.
  - PATTERN-lym-files holds: the new tracked files sit under `RunDir` and their locks under `ScratchDir`, as `StatusLock` does.
  - PATTERN-test-only-code holds: every new production function has a production caller.
  - PATTERN-tolerant-records holds: the writers go through `state.UpdateJSON` and the reads through `ParseStatusTolerant`.
  - PATTERN-refusal-way-forward is covered by F5.

## Proposed-fix check

Every fix that changes targets was overlaid together with `lym loom validate-plan --overlay`, against a `{}` baseline. That covers F3, F4, F5, F6, F9, F10, F12, F13, F14, F15, F16, F17 and F18. The combined run reported `ok: true` with no blocking finding.

## Rejected

- The affected-set lens listed `internal/landingshed#TestFixerAttempts` among card 13's missed members. It builds its paths from `t.TempDir()` and passes them to `FixerAttemptsPath`, which takes any directory, so it needs no change.
