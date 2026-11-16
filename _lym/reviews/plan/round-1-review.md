---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/06-run-reader-and-act-ask.md (Uses internal/battenrecipe#RowsBeforeRunShed)
    summary: pairteardown importing battenrecipe forms a test import cycle through preflightshed's in-package test, hubforge and pairteardown, so card 6 cannot build its own gate
    fix: In card 6, drop the Uses line `internal/battenrecipe#RowsBeforeRunShed`. Change the Intent so `ReadRun` holds the batten rows before Run-Shed as local constants in pairteardown, as `internal/pairteardown/followup.go` already holds `landRow` and `worktreeTeardownRow` for the same cycle. A pairteardown test pins them against `battenrecipe.RowsBeforeRunShed()`; pairteardown's tests may import battenrecipe, as followup_test.go does. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F2
    severity: BLOCKING
    class: scope
    origin: handler
    location: _lym/plan/07-pull-request-record-tracked.md (Intent; internal/landingshed/prsession.go:80)
    summary: Card 7 makes the PR record cross machines, but on another machine ensureSession resumes the recorded SessionID, which claudeengine's checkResumable refuses for lack of a transcript, halting PR-Review where D3 says the session starts fresh
    fix: Add `internal/landingshed#PRReview.ensureSession` to card 7's Edit list. In the Intent, `Record` gains `Machine` (JSON key `machine`), which every session-identity write sets to `machinename.Name()`. `ensureSession` sets `spec.ResumeSessionID` only when `rec.Machine` is this machine. A record written on another machine starts a fresh session and rewrites the identity through `updateRecord`. Add a tier 0 row to the test that drives `ensureSession`, in which a record naming another machine starts a session with no resume id. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/00-overview.md (Shared Decisions); cards 11 to 14
    summary: D1 and D3 steps no card carries and no Added-after-Discussion entry drops, so the parent must decide whether this part defers them
    fix: "The uncarried steps: D1's \"Every verb or loop that acts on a run asks the act rule first\", which leaves lym batten step, lym batten goto, lym loom start, lym shed step and lym batten cancel unguarded; D1's \"every reader of a run's state takes it from internal/runfsm\"; D1's board display written from the FSM state, which the overview keeps on its old writers; and D3's pause record entry written from the run's pause intent at an owner's catch-up. Ask the parent, with lym creel post --parent, to record with lym loom decision add that this part's act-rule asks are the four verbs Delivers names and that these steps belong to later parts. Then cite that entry in the overview's Shared Decisions. If the parent rules them in scope, add them to cards 12 and 13 (batten step, goto, loom start), 9 (cancel) and 14 (catch-up), each asking AskAct with the asides D1 and the shed-pause entry name."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/05-fabric-origin-run-reads.md (Intent of Topology.OriginParentHoldsTaskWork)
    summary: D1's rule that under the push skip the local parent branch decides past-merge is carried by no card, so under SkipPush a landing never reads past its parent-side merge
    fix: In card 5's Intent, `OriginParentHoldsTaskWork` reads the local parent branch through `ParentHoldsTaskWork` when the hub's sync options skip push (`SyncOptions.SkipPush`), since nothing reaches origin then. Otherwise it reads origin's fetched parent branch as written. `TestOriginParentHoldsTaskWork` gains a push-skip row in which a local parent merge reads true.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/11-dobby-asks-the-act-rule.md (Edit list); internal/battencli/dobbyreads.go:361
    summary: DobbyTasks sets a start refusal only for a done record, so a released task whose kept cancelled record stands is started every tick and each start is refused by card 12, piling into failing-pass notices
    fix: Add `internal/battencli#DobbyTasks` to card 11's Edit list. In the Intent, a released task whose batten record reads cancelled through `pairteardown.ReadRun` carries `StartRefusal` with `pairteardown.ActRefusal`'s cancelled wording, which names `lym batten cancel <slug> --remove`. The test that covers DobbyTasks' done-record start refusal gains a cancelled row. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/12-batten-run-asks-the-act-rule.md (refuseByAct cancelling branch)
    summary: refuseByAct's cancelling branch cannot be reached from lym batten run, because armAt's startGuard.refuse refuses a present cancel intent before the PreRun hook runs
    fix: "In card 12's Intent, state that a cancelling run is refused upstream by `startGuard.refuse`'s cancel check (card 9's `Cancel`), which `armAt` runs through `refuseAtStart` before `battenPreRun`. Drop \"a cancelling\" from the list of waits refuseByAct refuses, and drop the cancelling row from `TestRefuseByAct`. The cancelled row stays, since a finished cancel passes the guard."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/12-batten-run-asks-the-act-rule.md (Intent, landed verdict) and _lym/plan/00-overview.md (Shared Decisions)
    summary: Card 12 and the overview let a landed verdict through to the start guard, but armAt runs the start guard and the done-motif clear before battenPreRun, so the stated bound of that pass-through is false
    fix: "In card 12 and the overview's Shared Decisions bullet on `lym batten run`, replace the start-guard reasoning. By PreRun, armAt's `refuseAtStart` and `claimStart` have already cleared or refused a done batten record. A landed verdict there can only be a slug with no batten record whose board entry is done. Refuse that verdict through `ActRefusal`'s landed wording, so a done task is never seeded again, and change `TestRefuseByAct`'s landed row from passing to refusing."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-loom-resume-asks-the-act-rule.md (refuseByAct hand-over branch)
    summary: refuseOwnedElsewhere runs first and judges the owner against origin, so refuseByAct's hand-over from the local Bolt is reached only when that Bolt is stale, the one case in which the hand-over is wrong
    fix: In card 13's Intent, refuseByAct reads a hand_over verdict as act, because `refuseOwnedElsewhere` has already judged the owner against origin, or against the local Bolt offline. Every other non-act answer still refuses through `ActRefusal`. In `TestResumeVerb_RefusesWhatTheActRuleHolds`, replace the hand-over envelope row with a row where the fake askAct answers hand_over and the verb goes on to the run state.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-cancel-intent-replaces-marker.md (endRunPhase)
    summary: endRunPhase publishes the intent even when recordCancelIntent recorded nothing, and PublishBattenRecord then commits a run directory git never tracked, failing the cancel of a leftover pair with no records
    fix: In card 9's Intent, `endRunPhase` calls `PublishIntent` only when `RecordIntent` reports true. A slug with no batten record goes on to `EndForCancel` with nothing to publish. `TestCancelRun` gains a row where `RecordIntent` reports false, `PublishIntent` is not called, and the run is ended.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-run-reader-and-act-ask.md (ActRefusal wordings); _lym/plan/00-overview.md (Shared Decisions)
    summary: ActRefusal words the owed-push, code-tip and hold refusals, but AskAct keeps those inputs false in this part, so only a constructor-only test reaches them
    fix: "In card 6, drop the owed push, missing code tip and hold wordings from `ActRefusal` and their rows from `TestActRefusal`. `ActRefusal` then wraps any reason it does not word as a programmer error. In the overview, replace the Shared Decisions sentence \"ActRefusal already words their refusals so the later parts only fill the inputs\" with a line saying the part that fills each input adds its wording and its reaching test."
  - id: F11
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/08-fixer-attempts-tracked.md (Edit list)
    summary: Card 8 lists only the fixture constructors, but seven test members seed or read the attempt record through FixerAttemptsPath over ScratchDir and break once production reads RecordsDir
    fix: Add to card 8's Edit list `internal/landingshed#finalizeGateFixture.seed`, `internal/landingshed#finalizeGateFixture.attempts`, `internal/landingshed#TestFinalize_GateAndFixer`, `internal/landingshed#mergeInFixture.seedAttempts`, `internal/landingshed#mergeInFixture.attemptsOnDisk`, `internal/landingshed#TestMergeIn_Fixer` and `internal/landingshed#TestFixerRun_Outcomes`. The Intent says each builds the path from `RecordsDir`. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/09-cancel-intent-replaces-marker.md, 11-dobby-asks-the-act-rule.md, 08-fixer-attempts-tracked.md (doc text left stale)
    summary: The cards keep under Uses the package docs and file headers that describe the cancel marker and the scratch fixer record, so shedrun, pairteardown, dobbyengine, battencli/cli.go and fixerattempts.go keep text the cards falsify
    fix: Change three cards and the Uses lists of three more. Card 9 lists under Edit `internal/shedrun/doc.go`, `internal/pairteardown/doc.go` and `internal/battencli/cli.go` (its file header and the `preDrive` field doc name the cancel intent), and drops `internal/dobbyengine/doc.go` from Uses. Card 11 moves `internal/dobbyengine/doc.go` from Uses to Edit, so its planner, `strandedEnded`, reminder and notice text reads the cancelling Phase. Cards 4 and 7 drop `internal/shedrun/doc.go` from Uses, and card 6 drops `internal/pairteardown/doc.go`, which the uses-later-target check otherwise refuses. Card 8 replaces `internal/landingshed#FixerAttemptsPath` with `internal/landingshed/fixerattempts.go`, so its header and the path's doc both say the records directory; listing both is refused by redundant-file-target. Checked together with validate-plan --overlay; they add no blocking finding.
  - id: F13
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/07-pull-request-record-tracked.md (Edit list); internal/loomcli/pawlwiring_integration_test.go:88
    summary: Card 7 changes pawlRetiredMove's removal to publish and moves the record out of scratch, but omits TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord, which seeds and asserts it
    fix: Add `internal/loomcli#TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord` to card 7's Edit list. In the Intent, the test seeds the tracked record and asserts the removal was published. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/14-tender-run-pause-intent.md (Resume order); internal/tenderengine/resume.go:121
    summary: Resume clears and pushes the pause intent before the lift, which can still refuse a slug that only a hub-wide pause covers, so that refusal arrives after a write
    fix: In card 14's Intent, `Resume` runs the existing hub-wide-only refusal before `SetRunPause(slug, false)`. A slug with no entry of its own that a hub entry covers is refused with nothing written. `TestResume` gains that row, asserting `SetRunPause` was not called.
  - id: F15
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-cancel-intent-replaces-marker.md (refuseNothingLeft)
    summary: refuseNothingLeft counting a present intent never decides, because the intent sits in the status file inside the run directory, so RecordsPresent is already true whenever it is present
    fix: In card 9, drop the sentence that refuseNothingLeft counts a present intent as left. Change its re-sign arrow to `func refuseNothingLeft(slug string, remove, endOnly bool, rerun string, parts fabricengine.PairParts, deps cancelDeps) error`, dropping the reading parameter, and drop `reading.Present` from its left terms.
  - id: F16
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-dobby-asks-the-act-rule.md (planPass)
    summary: Once planPass gives no pass unless the verdict is act, and the act rule waits on the same pause record Covered reads, planPass's covered block is dead, and the card does not say it goes
    fix: In card 11's Intent, say that `planPass` drops its `covered` parameter's block, and that the lift's full pass comes from the uncovered branch through `trigger.Lift`. `TestPlanPass` drops its covered rows in favour of the wait-verdict rows the card already adds.
  - id: F17
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/10-cancel-finish-and-removal.md (TestCancelIntegration_IntentAndFinishReachOrigin)
    summary: The new integration test's keep-cancel origin assertions belong in TestCancelIntegration_KeptPairJudgeOnly, which card 10 already edits, and its pair-removed reading duplicates card 6's TestReadRun kept-cancel row
    fix: In card 10, put the assertion that the kept cancel's record holds the intent and its finish on origin into `TestCancelIntegration_KeptPairJudgeOnly`. Drop the ReadRun-with-the-pair-removed assertion, which `TestReadRun` covers. Keep the new test for the interrupted cancel reading cancelling on another machine, renamed `TestCancelIntegration_InterruptedCancelReadsCancellingElsewhere` in its Create target and Intent. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F18
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/02-runfsm-package.md (TestOrder)
    summary: TestOrder is a per-symbol test pinning a constant list whose precedence TestDerive's overlap rows already pin
    fix: In card 2, drop the Create target `plan:internal/runfsm#TestOrder` and its Intent line, and have `TestDerive` assert `Order()` beside its overlap rows. Checked with validate-plan --overlay; it adds no blocking finding.
  - id: F19
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/10-cancel-finish-and-removal.md (Teardown.dropCancelledRecord)
    summary: Card 10 names no lock for dropCancelledRecord's read and removal of the run directory, and does not say how it composes with the cancel's own --remove deletion, whose RemovePair reaches the same hook
    fix: In card 10's Intent, `dropCancelledRecord` holds the slug's `shedrun.StatusLock` across its read and removal, as the other writers of the record do. Under `--remove`, the cancel's `RemovePair` already drops the cancelled record, so the clear phase's `RemoveRecords` finds nothing tracked and commits nothing. `records_removed` reads true either way. `TestCancelRun` asserts that order.
---

### [BLOCKING] pairteardown cannot import battenrecipe

**Location:** `_lym/plan/06-run-reader-and-act-ask.md`, Uses `internal/battenrecipe#RowsBeforeRunShed`.

**Issue:** `go list -test -deps` over battenrecipe's dependencies shows `internal/preflightshed`, which battenrecipe reaches, building an in-package test.
That test, `preflight_integration_test.go`, imports `hubforge`, and hubforge reaches `pairteardown`.
A pairteardown import of battenrecipe closes the cycle.
`internal/pairteardown/followup.go:18-25` already records this exact cycle and holds row names locally for it.

### [BLOCKING] A tracked PR record resumes another machine's session

**Location:** `_lym/plan/07-pull-request-record-tracked.md`; `internal/landingshed/prsession.go:80`; `internal/shuttleengine/claudeengine/resume.go:52-58`.

**Issue:** `ensureSession` sets `spec.ResumeSessionID = rec.SessionID` whenever the recorded strand is not live, and on another machine it never is.
`checkResumable` refuses a session id with no transcript in this directory, and `ensureSession` returns any non-transient start failure as a `sessionStartHalt`.
D3 says a tracked session id whose transcript is absent here starts that session fresh.
Card 7 makes the record cross machines, but only edits `Record`'s doc to say the id holds on its writer alone.

### [MEDIUM] Decision steps carried by no card

**Location:** `_lym/plan/00-overview.md` Shared Decisions; cards 11 to 14.

**Issue:** Delivers names four verbs that ask the act rule.
D1 says every verb or loop that acts on a run asks, and the record's last entry names `lym batten step` and `lym loom start` among the verbs that set the shed pause aside.
No card guards batten step, batten goto, loom start, shed step or batten cancel.
The board's `<state> · <row>` display stays on its old writers, and no reader beyond the four moves to runfsm.
D3's catch-up pause-record entry has no card.
The overview defers some of these in Shared Decisions, but the decision record holds no entry dropping them, so the parent has to rule.

### [MEDIUM] The push skip of D1 is uncarried

**Location:** `_lym/plan/05-fabric-origin-run-reads.md`.

**Issue:** D1 says that under the push skip the local parent branch decides whether a landing is past its parent-side merge.
`SyncOptions.SkipPush` exists (`internal/fabricengine/fabric.go:81`).
Card 5 says the local parent branch never decides, with no exception.

### [MEDIUM] Dobby starts a released task whose cancelled record stands

**Location:** `_lym/plan/11-dobby-asks-the-act-rule.md`; `internal/battencli/dobbyreads.go:361-376`; `internal/dobbyengine/starts.go:68-75`.

**Issue:** `readyToStart` holds a task only on `StartRefusal`, pending dependencies, an owner elsewhere or a pause.
`DobbyTasks` sets `StartRefusal` only for a done record.
A kept cancel's record reads cancelled under D1, yet the dobby starts the task, and card 12 refuses every start.

### [MEDIUM] refuseByAct's cancelling branch in batten run is unreachable

**Location:** `_lym/plan/12-batten-run-asks-the-act-rule.md`.

**Issue:** `armAt` runs `refuseAtStart` (arm.go:347), which runs `startGuard.refuse`; its cancel check (startguard.go:281-286) refuses run, step and goto before the PreRun hook.
The card's test reaches the branch only through a fake.

### [MEDIUM] The landed pass-through's stated bound is false

**Location:** `_lym/plan/12-batten-run-asks-the-act-rule.md`; `_lym/plan/00-overview.md`.

**Issue:** The start guard and the done-motif clear run in `armAt` (arm.go:347-354), before `battenPreRun`.
So nothing after refuseByAct is a start guard that bounds a landed verdict.

### [MEDIUM] loom resume's hand-over branch fires only when wrong

**Location:** `_lym/plan/13-loom-resume-asks-the-act-rule.md`; `internal/loomcli/ownercheck.go:49-73,103-126`.

**Issue:** `refuseOwnedElsewhere` runs first and judges the owner from an origin reconcile.
`AskAct` reads the local Bolt.
The hand-over branch is therefore reached only when that Bolt disagrees with origin's judgment.

### [MEDIUM] Publishing an intent that was never recorded

**Location:** `_lym/plan/09-cancel-intent-replaces-marker.md`.

**Issue:** `recordCancelIntent` reports false for a slug with no batten record, yet `endRunPhase` records and publishes unconditionally.
In that case `PublishBattenRecord` commits the run directory's path.
For a path git never tracked, that commit is the "pathspec did not match" failure `internal/loomcli/wiring.go:96-107` describes.
`refuseNothingLeft` lets such a cancel through on its pair parts (cancel.go:675-677).

### [MEDIUM] Refusal wordings no input reaches

**Location:** `_lym/plan/06-run-reader-and-act-ask.md`.

**Issue:** `AskAct` keeps the owed-push, code-tip and hold inputs false in this part.
Only `TestActRefusal` reaches their wordings, and it calls the constructor alone, which does not reach a refusal.

### [MEDIUM] Card 8 misses the tests that build the attempt path

**Location:** `_lym/plan/08-fixer-attempts-tracked.md`; `internal/landingshed/finalize_gate_test.go:140,148`; `parentalign_fixer_test.go:49,57`; `fixerrun_test.go`.

**Issue:** These members call `FixerAttemptsPath` over `ScratchDir` themselves.
The fixture constructors the card lists do not fix them, and aliasing the two directories would void the card's own assertion that nothing lands under `ScratchDir`.

### [MEDIUM] Doc text the cards falsify stays under Uses

**Location:** cards 9, 11 and 8.

**Issue:** The following text names the cancel marker: `internal/shedrun/doc.go:51-52`, `internal/pairteardown/doc.go:154`, `internal/dobbyengine/doc.go`'s planner, ended, reminder and notice text, and `internal/battencli/cli.go:23,282`.
`internal/landingshed/fixerattempts.go:2` places the record in scratch.
No card lists any of them under Edit.

### [LOW] Card 7 omits the pawl move's test

**Location:** `_lym/plan/07-pull-request-record-tracked.md`.

**Issue:** The test seeds a scratch record and drives `pawlRetiredMove`, whose removal the card makes publish.

### [LOW] Resume writes before a refusal

**Location:** `_lym/plan/14-tender-run-pause-intent.md`.

**Issue:** `SetRunPause(false)` commits and pushes, and only then can `liftEntries` refuse the hub-wide-only slug (resume.go:121-124).

### [LOW] A term of refuseNothingLeft that never decides

**Location:** `_lym/plan/09-cancel-intent-replaces-marker.md`.

**Issue:** The intent is a key of `status.json` under the run directory, so `RecordsPresent` (cancel.go:658, 675) already holds whenever the intent does.

### [LOW] planPass's covered block becomes dead

**Location:** `_lym/plan/11-dobby-asks-the-act-rule.md`; `internal/dobbyengine/passes.go:144-152`.

**Issue:** With no pass unless act, and act requiring no covering pause, `covered` is false wherever the block is reached.

### [LOW] An integration test overlapping existing coverage

**Location:** `_lym/plan/10-cancel-finish-and-removal.md`.

**Issue:** `TestCancelIntegration_KeptPairJudgeOnly` already runs a keep cancel through hubforge with an origin, and card 10 edits it.
Card 6's `TestReadRun` already reads a kept cancel as cancelled on a machine without its pair.
Only the interrupted cancel reading cancelling elsewhere is behavior no other test covers.

### [LOW] TestOrder pins a constant per symbol

**Location:** `_lym/plan/02-runfsm-package.md`.

**Issue:** `Derive` takes the first state in `Order`, so `TestDerive`'s overlap rows already pin the precedence; a separate test of the list is one test per symbol.

### [LOW] dropCancelledRecord's lock and its composition with --remove

**Location:** `_lym/plan/10-cancel-finish-and-removal.md`.

**Issue:** The cancel's `RemovePair` seam is `teardown.RemovePair` (cancelcmd.go:286), so under `--remove` the hook drops the record before the clear phase's `RemoveRecords` runs.
The card names neither that order nor a lock for the hook's read and deletion.

## Rejected

- guard-reach's finding that card 3's `Shed.stepLocked` cancelled refusal is unreachable: the same lens names the generic `lym shed step` as a verb that steps a run without asking the act rule, and that verb reaches the engine's read gate. The engine guard is reachable, and its test drives the engine directly, which is where the guard lives.
- The handler's own candidate, that card 14's `RunAct` refuses a resume of a landed or cancelled run's pair: the acceptance criterion says `lym tender resume` refuses a run the act rule answers wait for, and D1 has terminal states wait. Not a defect.

## Settled without a finding

- Fidelity: D1's states, order, Derive, the queued reading of a leftover cancelled record and the act rule are on cards 2, 5 and 6. Its four act-rule asks are on cards 11 to 14. D3's cancel intent, finish, cancelled state, removal-time deletion and cleanup repair are on cards 3, 4, 9 and 10. The pause intent with its owner refusal is on card 14, the PR record on card 7, the fixer counts on card 8, and the cache rule on card 1. The origin-owed marker staying is licensed by Delivers. The uncarried steps are F2, F3 and F4.
- Card order: the plan-time order holds, as the gate's uses-later-target check confirms. F1 is a build-time cycle, not an order defect.
- Attack surface: cards 9, 10, 11, 13 and 14 state what each intent or aside lets through and its offline bound, the overview's D3 bound included. The card 12 landed pass-through is F7.
- Edit glyphs and re-sign arrows: the Edit cards list member glyphs, and the changed signatures carry arrows.
- Board entry: `lym board get` holds no entry for this slug. No card edits a board entry's body or scope, and no card files or promotes a task.
- PATTERN fit: the tracked records sit under `_lym` through shedrun and loomengine, the locks under `.lym` through `lock.TrackedFileLockPath` and `ScratchRoot`, and the git reads in fabricengine. That holds `PATTERN-lym-files` as declared and `PATTERN-fabric-boundary`.
- Research steps: no card leaves a fact to find while it runs. The batten record's fields and the existing push call sites are named on cards 3, 6, 7 and 9.
