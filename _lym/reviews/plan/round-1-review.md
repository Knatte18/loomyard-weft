---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/03-runfsm-reader-batten-run.md:8
    summary: >-
      Cards 3 and 5 together form an import cycle, dobbyengine to runfsm to hubgeom to dobbyengine, so card 5 cannot compile;
      card 3's reader imports hubgeom for ParentOf, hubgeom imports dobbyengine, and card 5 adds a runfsm.Records field to dobbyengine's RunView.
    fix: >-
      In card 3, re-sign the ReadOwner handle to
      `func ReadOwner(prime *lymcwd.Location, slug, here string, parentOf func(slug string) (string, error)) (Owner, error)`,
      drop `internal/hubgeom#ParentOf` from card 3's Uses,
      and say in its Intent that runfsm imports no hubgeom and that battenCLI.wire passes hubgeom.ParentOf over the prime.
      In card 10's Intent, have RunOwner pass the same parentOf when it calls runfsm.ReadOwner.
      In 00-overview.md's Shared Decision "runfsm's imports", remove hubgeom from the reader's imports
      and state that runfsm imports nothing that imports dobbyengine.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/08-cleanup-cancelled-record.md:16
    summary: >-
      Card 8 makes judgeStatus take a finished cancelled record as a candidate, and clearDoneRun uses judgeStatus as its done check;
      ClearDoneRun, the dobby's done-run clear, would then remove a kept cancel's record and discard its task branch on origin.
      That breaks D3 and the cancelled-run row card 7 adds to TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits.
    fix: >-
      In card 8, re-sign `internal/battencli#judgeStatus` to
      `func judgeStatus(data []byte, prefix, slug string) (candidate, cancelled, tracked bool, reason string)`
      and `internal/battencli#judgeRunDirectory` to
      `func judgeRunDirectory(location *lymcwd.Location, slug string, deps cleanupDeps) (candidate, cancelled, tracked bool, reason string)`.
      List `internal/battencli#clearDoneRun` under Edit.
      State in the Intent that candidate stays true for a done run only, that cancelled reports a finished cancel apart from it,
      that cleanupSlug removes a cancelled record only once its pair is gone,
      and that clearDoneRun keeps admitting done runs alone, so ClearDoneRun never takes a cancelled run.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/01-shedengine-cancelled-state.md:18
    summary: >-
      Card 1 searches only for case switches over the shed states, so it misses every reader that compares a state with done using == or !=.
      Once card 7 keeps a cancelled record, that record holds a dobby max_runs slot forever (dobbyengine/view.go:105),
      makes lym beam create treat the run as live (beamcli/create.go:202), and reads to adopt as a live run (adoptcli/adopt.go:320, :536).
    fix: >-
      In card 1, widen the Intent's search to every comparison with shedengine.StateDone and with the dobby's runStateDone,
      and give `cancelled` the finished-not-landed treatment at each hit.
      List under Edit
      `internal/dobbyengine#BuildView`, `internal/dobbyengine#daemon.drivePasses`, `internal/beamcli#createSteps`,
      `internal/adoptcli#movedRuns`, `internal/adoptcli#resumeSlug`, `internal/battencli#battenCLI.refuseAtStart`,
      `internal/battencli#childRunDone`, `internal/battencli#DobbyTasks` and `internal/pairteardown#BattenFollowUp`.
      Add `internal/beamcli/doc.go` and `internal/adoptcli/doc.go` under Uses.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/11-pull-request-record-tracked.md:58
    summary: >-
      Cards 11 and 12 leave the shared landing fixture envkit.LandingDeps without the new PullRequest.Publish seam and AttemptsDir,
      and leave batten's intentionallyNil map without the new nil Publish seam.
      Card 12's empty-AttemptsDir refusal then fails every envkit landing test, and the seam tests card 11 lists only under Uses fail on the new nil seam.
    fix: >-
      In card 11, list `internal/testkit/envkit#LandingDeps` and `internal/battencli#intentionallyNil` under Edit.
      In the Intent, say that LandingDeps fills Publish, that its RecordPath takes the record's new path,
      and that intentionallyNil gains Landing.PullRequest.Publish as a loom-only seam.
      In card 12, list `internal/testkit/envkit#LandingDeps` under Edit and say in the Intent that it fills AttemptsDir.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/12-fixer-attempts-tracked.md:23
    summary: >-
      Card 12's batten changes rest on a false premise: batten builds no landingshed.Deps and writes no fixer attempts.
      env.Landing is filled only by loomcli (sharedbootstrap.go:399), so wiring battenCLI.wire to FixerAttemptsDir and staging FixerAttemptsRel in battenRunCommitPaths has no substrate.
    fix: >-
      In card 12, drop `internal/battencli#battenCLI.wire`, `internal/battencli#battenRunCommitPaths` and `internal/battencli#TestNewCommitStatusSeam` from Edit.
      Strike their sentences from the Intent, so the counts are committed only through loomcli's statusCommitPathspec in the pair.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-cancel-finish-keeps-record.md:31
    summary: >-
      Under --remove, card 9's RemovePair already deletes and commits the batten record and spawns a detached push.
      Card 7's following RemoveRecords then commits nothing, so the synchronous publish card 7 promises has nothing to push,
      and the removal reaches origin only through the detached push.
    fix: >-
      In card 7's Intent, have clearRunPhase under --remove call the FinishRecord seam's PublishIntents after removePairAndBranch and RemoveRecords
      whether or not RemoveRecords committed, so the commit card 9 made is pushed synchronously before the cancel returns.
      Have TestCancelIntegration_RemoveTearsDownPairAndBranch assert the record is gone from origin when the verb returns.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/05-dobby-pass-asks-act-rule.md:23
    summary: >-
      Card 5 drops planPass's run.CancelMarker check, saying the verdict covers it, but the verdict reads only the pushed intent.
      So the dobby spawns a pass during a cancel whose intent is not yet committed, the window the Shared Decision says the local lock covers,
      and that pass refuses and counts as a failing pass.
    fix: >-
      In card 5's Intent, keep planPass's run.CancelMarker check, which card 6 fills from the intent and the held cancel lock, beside the act rule's verdict.
      Drop the sentence that removes it, and keep only the removal of drivable's checks the verdict covers.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-loom-resume-asks-act-rule.md:18
    summary: >-
      resumeCmd runs refuseOwnedElsewhere through runOwnerVerdict before resumeFromRunState, so card 4's act check can never answer hand_over in production.
      Filling Inputs.OwnedElsewhere from a second runOwnerVerdict adds a second origin reconcile to every resume, and its TestResumeVerb row pins a state the verb cannot reach.
    fix: >-
      In card 4's Intent, leave Inputs.OwnedElsewhere at its zero value, since resumeCmd's refuseOwnedElsewhere has already refused a run owned elsewhere.
      Drop the sentence about keeping the StartOwnedElsewhereKind envelope for a hand_over.
      Cut TestResumeVerb's owned-elsewhere row, keeping the cancelling and shed-paused rows.
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/11-pull-request-record-tracked.md:64
    summary: >-
      Card 11 leaves whether PRReview.resumeSession already falls back to a fresh session to be found out at run time ("grep it first and add only the fall-back that is missing"),
      and names no test that reaches the fresh-start branch.
      resumeSession (prsession.go:217-228) calls ensureSession with no fall-back.
    fix: >-
      In card 11's Intent, replace the grep instruction with the researched fact:
      resumeSession calls ensureSession on the recorded session with no fall-back, so the card adds the branch that starts the session fresh
      when the engine's resume check finds the transcript absent here.
      Name the input that reaches it (a record whose session id has no transcript on this machine)
      and the test, a new row in TestPRReview_Session.
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/06-cancel-intent-replaces-marker.md:17
    summary: >-
      Card 6 has takeCancelMarker write CancelIntent{StartedAt, Machine, Remove} from the --remove flag,
      but takeCancelMarker takes only (location, slug, wait), hubCancelDeps has no remove,
      and cancelDeps.TakeMarker is a no-argument seam; the card gives neither a re-sign arrow nor the cancelDeps edit.
    fix: >-
      In card 6, write the takeCancelMarker bullet as
      `internal/battencli#takeCancelMarker` -> `func takeCancelMarker(location *lymcwd.Location, slug string, remove bool, wait time.Duration) (heldCancelMarker, error)`
      and list `internal/battencli#cancelDeps` under Edit.
      Say in the Intent that cancelDeps.TakeMarker takes remove, which endRunPhase passes.
  - id: F11
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/02-runfsm-core.md:63
    summary: >-
      Card 2 and the Shared Decision let PastParentSideMerge ignore the states' precedence, so Act answers act for a paused or cancelling run past its merge.
      D1 grants that act only to a run whose state is landing, and no Added after Discussion entry records the departure.
    fix: >-
      Either restrict card 2's act-past-merge line to a run whose Derive state is landing, as D1 reads,
      or have the parent record the shed-paused departure with lym loom decision add and cite that entry in the Shared Decision "Past the parent-side merge".
      In either case, state that a cancelling, landed or cancelled run past its merge still answers wait.
  - id: F12
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/00-overview.md:46
    summary: >-
      Decision steps no card carries and no Added after Discussion entry records as dropped:
      D1's board status written from the FSM state (the overview keeps it written as today);
      D3's fresh start of a Claude session id in webster's state whose transcript is absent here;
      D3's pause record entry written from the pause intent at the owner's catch-up;
      and D3's hand-over notice naming each dropped intent.
    fix: >-
      Carry the webster session-id fresh start in a card beside card 11's PR-Review fall-back.
      For the board-status display, the catch-up entry and the hand-over notice, either carry each in a card
      or ask the parent to record with lym loom decision add that the later part delivering D4 and D6 carries it.
      Then name that entry in 00-overview.md's Shared Decisions "Board status" and "Inputs left at zero".
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/05-dobby-pass-asks-act-rule.md:12
    summary: >-
      The cards repeatedly leave behind doc and help text that their own change makes false:
      dobbyengine/doc.go's pass rules (card 5); battencli/cli.go's header and registeredBattenAlive's comment on the marker (card 6);
      cancelCmd's help and cancelEnvelope, which never renders the finish (card 7); cleanupCmd's help (card 8);
      and pullrequest/doc.go, pairteardown/doc.go and RemovePair's comment on the scratch record (card 11), where card 11 lists pullrequest/doc.go under Extends while rewriting it.
    fix: >-
      Card 5: move `internal/dobbyengine/doc.go` from Uses to Edit.
      Card 6: add `internal/battencli/cli.go` and `internal/pairteardown#Teardown.registeredBattenAlive` under Edit.
      Card 7: add `internal/battencli#battenCLI.cancelCmd` and `internal/battencli#cancelEnvelope` under Edit, the envelope rendering the finish.
      Card 8: add `internal/battencli#battenCLI.cleanupCmd` under Edit.
      Card 11: move `internal/pullrequest/doc.go` from Extends to Edit,
      and add `internal/pairteardown/doc.go` and `internal/pairteardown#Teardown.RemovePair` under Edit.
      Each card's Intent names the text it corrects.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-shedengine-cancelled-state.md:18
    summary: >-
      lym batten run of a kept cancelled record stops at checkRunStatus before the act check;
      with the StateDone branch card 1 prescribes, it answers doneSlugRefusal's already-completed wording, never the act rule's lym fabric remove way forward.
    fix: >-
      In card 1, list `internal/battencli#battenCLI.checkRunStatus` under Edit.
      State in the Intent that a `cancelled` status refuses there naming `lym fabric remove <slug>` as the way forward,
      with a TestBattenRun or checkRunStatus row reaching it.
  - id: F15
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/02-runfsm-core.md:77
    summary: >-
      Card 2 creates Aside.Cancel and Aside.Hold for lym batten cancel, but no card makes the cancel ask the act rule,
      and the Goal's Delivers does not list it, so both fields are reachable only from TestAct.
    fix: >-
      In card 2's Intent, drop Aside.Cancel and Aside.Hold, together with their TestAct rows, keeping OwnPause and ShedPause,
      which cards 3, 4 and 10 set.
  - id: F16
    severity: LOW
    class: decision
    origin: handler
    location: _lym/plan/02-runfsm-core.md:53
    summary: >-
      Card 2 does not say what Derive returns when no state's condition holds.
      That case occurs: a draft entry after cancel --remove clears the claim, or arm's run-form claim before batten seeds its record.
      The answer feeds Act and is left to the implementer.
    fix: >-
      In card 2's Intent, name Derive's answer for records that meet no state's condition,
      for example an empty State that Act answers act for and Verdict.Refusal never names.
      Add TestDerive rows for a draft board status with no batten record and for a run-form status with no batten record.
  - id: F17
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-dobby-pass-asks-act-rule.md:26
    summary: >-
      TestPlanPass's row "a run past its merge under a hold gets a pass" cannot be reached through planPass, since card 5 leaves the hold input at zero and counts run.Held as a covering pause;
      TestAct already pins the past-merge-under-hold answer.
    fix: In card 5's Intent, cut that TestPlanPass row.
  - id: F18
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-cancel-intent-replaces-marker.md:74
    summary: >-
      TestCancelIntegration_AnInterruptedCancelReadsCancellingOnAnotherClone stops a cancel after its intent is published,
      but cancelRun has no seam a real-hub test can stop at, and the card names no input that brings the run to that state.
    fix: >-
      In card 6's Intent, have the test build the interrupted cancel on the first clone by writing and publishing an intent with no finish
      through runfsm.UpdateIntents and runfsm.PublishIntents, then read cancelling through runfsm.Read on the second clone.
      Keep the assertion that a full lym batten cancel leaves its intent on origin.
  - id: F19
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/07-cancel-finish-keeps-record.md:5
    summary: >-
      Card 7's new tier1 test TestCancelIntegration_AKeptCancelLeavesItsFinishOnOrigin covers the kept-cancel behaviour
      TestCancelIntegration_KeptPairJudgeOnly already drives over a hubforge hub, and card 7 already edits that test.
    fix: >-
      In card 7, drop the Create handle of TestCancelIntegration_AKeptCancelLeavesItsFinishOnOrigin.
      Put its assertions into TestCancelIntegration_KeptPairJudgeOnly:
      the finish and state cancelled on origin's records branch, and cancelled with keep through runfsm.Read on a second clone.
---

### [BLOCKING] Import cycle between dobbyengine, runfsm and hubgeom

**Location:** `_lym/plan/03-runfsm-reader-batten-run.md:8`, `_lym/plan/05-dobby-pass-asks-act-rule.md:20`, `_lym/plan/00-overview.md:34`

**Issue:**
- `go list -f '{{join .Imports "\n"}}' ./internal/hubgeom` lists `internal/dobbyengine`, and `internal/hubgeom/dobbygeom.go:29` returns `dobbyengine.Geometry`.
- Card 3's reader imports `hubgeom` for `ParentOf`, per the Shared Decision and card 3's ReadOwner paragraph.
- Card 5 gives `dobbyengine.RunView` a `runfsm.Records` field, so `dobbyengine` imports `runfsm`.

The build then cycles `dobbyengine` → `runfsm` → `hubgeom` → `dobbyengine`.
The Shared Decision's claim that nothing runfsm imports depends on it is false for `hubgeom`.
Every other planned import (`fabricengine`, `boardengine`, `shedrun`, `shedengine`, `state`, `lymcwd`) has no path back to `dobbyengine`, `pairteardown` or `tendercli`, so dropping `hubgeom` alone breaks the cycle.
The repo-invariants lens reported the same cycle.

### [BLOCKING] Card 8's cancelled candidate reaches the dobby's done-run clear

**Location:** `_lym/plan/08-cleanup-cancelled-record.md:16`

**Issue:**
`internal/battencli/cleanup.go:222` has `clearDoneRun` call `judgeStatus` to decide whether a run is done.
`ClearDoneRun` (`cleanup.go:139-150`) then discards the task branch locally and on origin (`cleanup.go:237-241`) and removes the records.
Card 8 widens `judgeStatus` to admit a finished `cancelled` record, so the dobby's done-run clear would take every kept cancel, branch included.
That is what D3 forbids ("never takes a cancelled run"), and it contradicts card 7's own premise and its new `TestClearDoneRun` row.
All three lenses reported this.

### [MEDIUM] Card 1 misses every equality reader of the done state

**Location:** `_lym/plan/01-shedengine-cancelled-state.md:18`

**Issue:**
Card 1's search covers `case shedengine.StateDone` switches only.
The readers that compare states with equality instead:
- `internal/dobbyengine/view.go:105-106` drops only `runStateDone` runs from the slot count and the view.
- `internal/beamcli/create.go:202` sets `RunLive` for any state but done.
- `internal/adoptcli/adopt.go:320` and `:536`, `internal/battencli/startguard.go:490`, `internal/battencli/dobbyreads.go:366`, `internal/battencli/readoptstop.go:303` and `internal/pairteardown/followup.go:35` judge done the same way.

Each confirmed span holds its line through `lym quarry resolve`.
Once card 7 leaves kept `cancelled` records, these sites read them as live runs.

### [MEDIUM] Shared landing fixture and batten's nil-seam map not updated

**Location:** `_lym/plan/11-pull-request-record-tracked.md:58`, `_lym/plan/12-fixer-attempts-tracked.md:54`

**Issue:**
`internal/testkit/envkit/envkit.go:181-253` (`LandingDeps`) builds the landing fixture with `ScratchDir` only and a `PullRequest` block without `Publish`.
`internal/battencli/wire_seams_test.go:38` (`intentionallyNil`) lists each `Landing.PullRequest.*` seam batten leaves nil.
Card 11 adds a func seam and card 12 a required directory, yet both list envkit's tests only under Uses.
Both lenses that reported it read the seam test's nil-field check.

### [MEDIUM] Card 12's batten wiring has no substrate

**Location:** `_lym/plan/12-fixer-attempts-tracked.md:23`

**Issue:**
`grep` finds `landingshed.Deps{` built only in `internal/loomcli/landingdeps.go:101` and envkit.
`env.Landing` is assigned only at `internal/loomcli/sharedbootstrap.go:399`.
`internal/battencli/wire.go` sets no `Landing`, and no battencli or battenshed file calls `FixerAttemptsPath`.
The batten edits therefore change nothing the run reads, and they stage a directory batten never writes at the prime.

### [MEDIUM] The --remove record removal is pushed only by card 9's detached spawn

**Location:** `_lym/plan/07-cancel-finish-keeps-record.md:31`, `_lym/plan/09-teardown-deletes-cancelled-record.md:23`

**Issue:**
`cancel.go:717` calls `deps.RemovePair`, which `cancelcmd.go:286` wires to `teardown.RemovePair`.
Card 9 deletes and commits the record there and spawns `SpawnStatusPush`.
`removeRunRecords` (`cleanup.go:482-500`) then finds the run untracked and commits nothing.
Card 7's synchronous publish therefore never carries the removal, contrary to D3's "committed and pushed".

### [MEDIUM] The dobby loses the local cancel lock in the uncommitted-intent window

**Location:** `_lym/plan/05-dobby-pass-asks-act-rule.md:23`

**Issue:**
`planPass` (`internal/dobbyengine/passes.go:123-127`) refuses a pass on `run.CancelMarker`.
Card 6 fills that field from the intent and the held lock.
Card 5 removes the check because the verdict "covers" it, but the verdict reads only the committed intent.
The Shared Decision's local-lock guarantee then no longer reaches the dobby.

### [MEDIUM] Card 4's hand_over branch is unreachable and doubles the reconcile

**Location:** `_lym/plan/04-loom-resume-asks-act-rule.md:18`

**Issue:**
`internal/loomcli/resume.go:276` refuses through `refuseOwnedElsewhere` before line 290 reaches `resumeFromRunState`.
`refuseOwnedElsewhere` (`ownercheck.go:103-126`) uses the same `runOwnerVerdict`, which runs a judge-only origin reconcile (`ownercheck.go:60`).

### [MEDIUM] Card 11 carries a research step and an untested branch

**Location:** `_lym/plan/11-pull-request-record-tracked.md:64`

**Issue:**
The card tells the implementer to grep for an existing fall-back and add only what is missing, which leaves a design fact to run time.
`internal/landingshed/prsession.go:217-228` shows `resumeSession` reading the record and calling `ensureSession` with no fall-back.
The new branch also names no reaching test, which the guard-reach lens reported too.

### [MEDIUM] takeCancelMarker's new inputs have no re-sign and no seam change

**Location:** `_lym/plan/06-cancel-intent-replaces-marker.md:17`

**Issue:**
`internal/battencli/cancelmarker.go:130` declares `takeCancelMarker(location, slug, wait)`.
`internal/battencli/cancelcmd.go:178` declares `hubCancelDeps(location, slug)`, and its TakeMarker closure (`cancelcmd.go:252-258`) has no remove to pass.
`cancelDeps.TakeMarker` (`cancel.go:63`) is `func() (...)`, so the flag must arrive through a signature change the card does not list.
The gate admitted the proposed arrow and the cancelDeps edit together with every other fix in this review.

### [MEDIUM] Act-past-merge applies to states D1 does not grant it to

**Location:** `_lym/plan/02-runfsm-core.md:63`, `_lym/plan/00-overview.md:43`

**Issue:**
D1 says `act` when the run is `landing` past its parent-side merge, and `wait` when the state is `paused` or `cancelling`.
Card 2 and the Shared Decision make PastParentSideMerge ignore precedence, so the dobby acts on a shed-paused run past its merge.
That may be the right call, but it overrides D1's text, and the record has no Added after Discussion entry for it.
The "A shed pause is an aside" entry names only lym batten run, step, loom start, loom resume, batten cancel and tender resume, never the dobby.

### [MEDIUM] Decision steps carried by no card

**Location:** `_lym/plan/00-overview.md:46`

**Issue:**
- D1 says the board's `<state> · <row>` status is written from the FSM state, but the overview keeps writing it as today.
- D3 says a webster-state session id whose transcript is absent starts fresh, but only the PR record's session is handled (card 11).
- D3's catch-up pause entry and the hand-over notice naming dropped intents appear in no card.

The Goal's Delivers is narrower, but the rubric accepts an uncarried Decision step only through an Added after Discussion drop, and the record holds none for these.

### [MEDIUM] Doc and help text left false by the cards' own changes

**Location:** `_lym/plan/05-dobby-pass-asks-act-rule.md:12`

**Issue:**
The text each card leaves false:
- `internal/dobbyengine/doc.go:153-175` describes the marker and the reached-parent pass rules card 5 replaces.
- `internal/battencli/cli.go:20-24` and `internal/pairteardown/teardown.go:711-715` describe the marker card 6 removes.
- `internal/battencli/cancelcmd.go:54-78` says the cancel removes the records and leaves its marker, and `cancelEnvelope` (`cancelcmd.go:150-175`) has no field for the finish card 7 reports.
- `internal/battencli/cleanup.go:628-637` says cleanup removes done runs only.
- `internal/pullrequest/doc.go:4` says the record lives in scratch and is never tracked, yet card 11 lists the file under Extends.
- `internal/pairteardown/doc.go:101-102` and `teardown.go:406-408` say the same.

### [LOW] A kept cancel never reaches the act rule's way forward through lym batten run

**Location:** `_lym/plan/01-shedengine-cancelled-state.md:18`

**Issue:**
`checkRunStatus` (`internal/battencli/arm.go:522-540`) runs before card 3's act check.
It refuses an unknown state with no way forward, or a done one with `doneSlugRefusal` (`arm.go:848-853`), which names cleanup and a branch delete.

### [LOW] Aside.Cancel and Aside.Hold have no setter

**Location:** `_lym/plan/02-runfsm-core.md:77`

**Issue:**
The cancel's own path (cancel.go, cancelcmd.go) never calls the act rule in any card, so these two fields serve only TestAct.
The guard-reach lens reported the same.

### [LOW] Derive's answer for unmatched records is unspecified

**Location:** `_lym/plan/02-runfsm-core.md:53`

**Issue:**
`clearRunClaim` (`cancelcmd.go:515-528`) writes `draft`, and `claimBoardEntry` (`owner.go:214`) writes a run-form claim before the batten record exists.
Neither meets any of the nine conditions card 2 lists.

### [LOW] TestPlanPass hold row is unreachable

**Location:** `_lym/plan/05-dobby-pass-asks-act-rule.md:26`

**Issue:**
Card 5 leaves the hold input at zero and counts `run.Held` as a covering pause, so through planPass a run under a hold gets `wait`.

### [LOW] The interrupted-cancel integration test has no way to stop the cancel

**Location:** `_lym/plan/06-cancel-intent-replaces-marker.md:74`

**Issue:**
`cancelRun` (`cancel.go:242-263`) runs from intent to end in one call, with no seam a hubforge test can interrupt.

### [LOW] Card 7 adds a test for a behaviour an existing test drives

**Location:** `_lym/plan/07-cancel-finish-keeps-record.md:5`

**Issue:**
`internal/battencli/cancel_integration_test.go:174-204` already runs a kept cancel at tier1 over a copied hub and asserts its envelope, and card 7 edits that test.
Its new assertions belong there.

## Rejected

- guard-reach #9 (lym tender resume of an owner-paused slug is refused with a way forward that does not lift the entry).
  Refusing it is D1's `hand_over`.
  The second way forward, `lym adopt` here, works: `internal/adoptcli/pause.go:115` turns the `OwnerCaller` entry into an adopt entry, which the resume then lifts once the run is owned here.
- repo-invariants #10 (status.go:13's "six legal State values" comment).
  Card 1's Intent already prescribes the seven-value count.
  No member span holds line 13, and the file self glyph beside card 1's `State.valid` would trip `redundant-file-target`, so no admissible target adds anything.
- affected-set #5's test readers of the record path (`pawlwiring_integration_test.go:88`, `commitrecords_test.go:197`, `pawlengine/retiredmove_test.go:57`).
  Card 11's Intent already says to grep for the old scratch path in tests and move each hit, and the card's gate fails on any it misses, so no plan change follows.
