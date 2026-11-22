---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/pullrequest/record.go#ClearReviewSession
    summary: "A goto onto PR-Review keeps ParentReviewAsked, so a review re-entered under autonomy never sends its first PR-Review ask."
    fix: "In internal/pullrequest/record.go ClearReviewSession, also set rec.ParentReviewAsked = false next to Autonomy and the session fields, and update its doc comment to list the field. Add a row to TestClearReviewSession in internal/pullrequest/record_test.go whose held record has ParentReviewAsked true and whose cleared record has it false."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/beamcli/dobbydeps.go:186
    summary: "Dobby's run notices (dead driver, usage limit, API-error stall, quiet child) post through the unstamped orchcli.NotifyRunFrom, so they carry no autonomy line where D2 asks every run notice an orch must settle to carry the run's value."
    fix: "In internal/orchcli/notify.go export withRunAutonomy as WithRunAutonomy and keep NotifyRunFromHalt as it is. In internal/beamcli/dobbydeps.go, build the line as orchcli.WithRunAutonomy(worktree, slug, line) inside runNotices.Post and keep geom.Slug as the signer scope passed to NotifyRunFrom, since NotifyRunFromHalt signs unscoped and would drop the beam dobby's scope. Extend dobbyengine's run-notice tests or beamcli's dobby wiring test with a row asserting the posted line ends in \"; autonomy: true\" or \"; autonomy: false\"."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/battencli/notices.go:27
    summary: "Only the helper that stamps a halt notice is tested; no test fails if the converted call sites (batten notifyRunParent, the loom hand-back Notify, the pre-push notify) go back to the unstamped NotifyParent."
    fix: "Add one row per converted seam asserting the posted line ends in \"; autonomy: true\" or \"; autonomy: false\": in internal/battencli (notifyRunParent for a present pair and for an absent pair), in internal/loomcli beside TestRunAutonomy_ForcesTheRunsKeysOverARealHub or the hand-back wiring test (drivewiring.go HandBack.Notify), and for newPrePushCheck's notify. Reuse the creel-on-fake-delivery harness TestNotifyHalt_CarriesTheRunsAutonomy uses; add no new harness."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/arm.go:479
    summary: "Nothing pins the three places that hand the PR-Review reset to a goto (arm.go spec.GotoAdmitted, arbiterconsume.go and drivewiring.go Admitted); removing any one fails no test and the D4 acceptance criterion that a goto onto PR-Review clears the record's autonomy and session goes unmet."
    fix: "Add one row to a loomcli test that builds the loom spec through specFor(\"goto\") on a receiver whose location is set and asserts spec.GotoAdmitted is non-nil and, called with loomshed.NamePRReview over a record holding autonomy and a strand, clears them. Add the same non-nil assertion for the RelaunchRequest that loomConsumeAttempt and goDriverDepsFor build (their Admitted field)."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/battencli/drive.go:602
    summary: "Batten's goto spec and its two RelaunchRequests (battenConsumeAttempt, ConsumeTackler) pass no Admitted hook, so a batten goto onto a halted PR-Review row keeps the recorded autonomy and session."
    fix: "Pass a PR-Review reset as Admitted on the batten goto spec (battencli/arm.go) and on the RelaunchRequests at drive.go:320 and drive.go:602, built over the run's own record and records committer as loomcli's prReviewGotoAdmitted is. When batten can never target PR-Review because haltedRowGuard admits only the halted row, instead state that in the doc comment of haltedRowGuard and leave the code."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go#TestPRReview_Wait
    summary: "Three added failure paths have no row: LoadConfig refusing a non-boolean autonomy, judgeParentAnswer failing on a nil or erroring PRReviewStanding, and Admitted returning an error from Goto and Relaunch."
    fix: "Add one row each to the test that already covers the behavior: a non-boolean autonomy in the resolved file to TestLoadConfig in internal/dobbyengine, a PRReviewStanding that returns an error and one left nil under a recorded autonomy to TestPRReview_Wait, and an Admitted that returns an error (the call's error comes back with the move already written) to TestGoto_Guard and TestRelaunch in internal/shedengine."
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/pawlwiring.go:152
    summary: "The Beam-Await drain wiring (HoldsCommits and WorkRemains over the board's open members, drafts included, and the beam's child pairs) has no test, so a miswired closure passes every row, which only drive fakes."
    fix: "Add a hub-level row over hubforge in internal/loomcli (beside TestRunAutonomy_ForcesTheRunsKeysOverARealHub) that builds pawlDeps for a beam and asserts WorkRemains is true with an open draft member, true with a child pair, and false with neither, and HoldsCommits is false before and true after a commit on the beam."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:948
    summary: "The Slack section still says the orch's turn-end hook posts each turn's final reply to its channel, which is wrong while autonomy is on, and the Beam-Await description omits the [beam drained] line."
    fix: "In docs/overview.md line 948 add that while the orch's autonomy is on the hook posts only a reply opening with [operator]. In the Beam-Await description add one sentence that an autonomous beam's row posts one [beam drained] line to its orch once the beam holds commits beyond its parent and no member, child pair or ready record stands."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: internal/orchengine/prompt_test.go#TestWithLaunchFacts
    summary: "TestWithLaunchFacts is listed as contained, but it pins what its covering test never checks: the pointer stays one line and the autonomy line stands with no presence path."
    fix: "Add a line //testtiming:keep pins the pointer's one-line shape and the autonomy line with no presence path directly above func TestWithLaunchFacts in internal/orchengine/prompt_test.go."
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: internal/orchengine/watcher_test.go#TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped
    summary: "The test is listed as contained, but its rows for a mark after the boundary and a mark with other text on the auto-reload path are asserted by none of its covering tests."
    fix: "Add a line //testtiming:keep pins the auto-reload path's skip of a mark after the boundary and its typing of a mark with other text directly above func TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped in internal/orchengine/watcher_test.go."
---

### [MEDIUM] A goto onto PR-Review leaves the review's first ask unsent

**Location:** internal/pullrequest/record.go#ClearReviewSession
**Issue:** `ClearReviewSession` clears the record's autonomy and session fields but keeps `ParentReviewAsked`, which only `settleSendBack` resets (`internal/landingshed/prreview.go:124`).
A run moved back onto PR-Review by `lym loom goto` records autonomy true anew and starts a new session, but `askParentReview` returns at once on `ParentReviewAsked`, so no first PR-Review ask goes out.
The autonomous prompt asks only after each set of fixes, so a session with nothing to fix has no ask to wait on, and the review stalls until a hold notice reaches the parent.
D4 says the re-entry starts a new session, and a re-entered review sends a new first ask.

### [MEDIUM] Dobby's run notices carry no autonomy

**Location:** internal/beamcli/dobbydeps.go:186
**Issue:** D2 asks every notice an orch must settle for a run to carry that run's autonomy, so the orch handles it under the run's value.
Dobby's run notices (a dead driver, a usage limit, an API-error stall, a quiet child) are the ones an orch resumes the run for.
They post through `orchcli.NotifyRunFrom`, unstamped, while batten's notices were moved to the stamped variant.
`NotifyRunFromHalt` is no drop-in here, since it signs unscoped and the dobby's own scope would be lost.
The other unstamped posters I read (stencils-changed, config-change, sketch-finish, raddle-filed and Discussion notices) are informational and are not part of this finding.

### [MEDIUM] The halt-notice stamp is pinned only at its helper

**Location:** internal/battencli/notices.go:27
**Issue:** `TestNotifyHalt_CarriesTheRunsAutonomy` covers `NotifyParentHalt` and `NotifyRunFromHalt`.
Nothing covers the call sites the diff converted, so a revert to `NotifyParent` at any of them fails no test, and D2's acceptance criterion that a run's halt notice carries its autonomy goes unpinned where it matters.

### [MEDIUM] The goto hook's wiring is untested

**Location:** internal/loomcli/arm.go:479
**Issue:** `prReviewGotoAdmitted` and `clearPRReviewOnGoto` are tested, but the three places that hand the hook to a goto are not: `arm.go`, `arbiterconsume.go` and `drivewiring.go`.
Dropping any one leaves the goto keeping the old autonomy and session, against D4's acceptance criterion.

### [LOW] Batten's goto and relaunch paths pass no Admitted hook

**Location:** internal/battencli/drive.go:602
**Issue:** The loom paths pass the PR-Review reset, but batten's goto spec and its two `RelaunchRequest`s do not.
`haltedRowGuard` limits a batten goto to the halted row, so the path is narrow: only a run halted at PR-Review is affected.
A run moved that way keeps its recorded autonomy and resumes its old session.

### [LOW] Three failure paths have no test row

**Location:** internal/landingshed/prreview_wait_test.go#TestPRReview_Wait
**Issue:** No row refuses a non-boolean resolved `autonomy` through `LoadConfig`, makes `judgeParentAnswer` fail on a nil or erroring `PRReviewStanding`, or has `Admitted` return an error from `Goto` or `Relaunch`, though the doc comment states the last one.

### [LOW] The drain wiring is untested

**Location:** internal/loomcli/pawlwiring.go:152
**Issue:** D6's conditions that drafts count and that a child pair blocks are decided by the `HoldsCommits` and `WorkRemains` closures, and only fakes drive them.
I read `ReadBeamLandBlockers` and it counts every non-done task and beam project of the beam plus its child pairs, so the logic matches D6; the gap is coverage only.

### [LOW] docs/overview.md leaves two behaviors undescribed

**Location:** docs/overview.md:948
**Issue:** The Slack hook description is wrong under autonomy, and the Beam-Await description omits the drain notice.
The same diff edits the PR-Review and `parent-review` passages of that file, so the doc set for this change is partly updated.

### [LOW] TestWithLaunchFacts pins what its covering test does not

**Location:** internal/orchengine/prompt_test.go#TestWithLaunchFacts
**Issue:** The redundancy report lists it as contained in `TestResumeContext_RendersPointerPerPhaseAndWritesMark`.
That test checks neither the one-line shape of the pointer nor the autonomy line with no presence path, so the test is kept and marked.

### [LOW] The hook-delivery test pins rows its covering tests do not

**Location:** internal/orchengine/watcher_test.go#TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped
**Issue:** The covering tests have no row for a mark after the boundary or for a mark with other text on the auto-reload path, so the test is kept and marked.

## Rejected

- Lens code-affected-set F3 (`contracts/orchrole/role-beams.md` and `plugins/ly/skills/board/SKILL.md` do not name `autonomy`): neither file lists the per-task beam settings; each names `land_notice` only in a sentence about the landing notice, so nothing there is stale.
- Lens code-correctness F2 (the retry way forward of a failed goto clear may not re-run `Admitted`): unconfirmed, and `shedengine.Goto` runs `Admitted` after any move it writes, including onto the current row; only `Relaunch` skips it, and the retry names `lym loom goto`.
- Lens code-scope F1 and code-affected-set F1 for the sites other than dobby's run notices (`loomcli/cli.go:211`, `landingdeps.go:279`, `wiring.go:267` and `wiring.go:995`, `raddlewiring.go:246`, `raddleverbs.go:248`, `darnwiring.go:272`): read at their call sites, they post informational notices (stencils changed, config change, sketch finished, Discussion lines, raddle filed, verify budget), not a halt of a run an orch settles.
- Lens code-correctness note that clearing an ask and dying before the new ask is sent spawns a judge on the next resume: the fork reports nothing is lost, and D3 asks only that a resume re-ask, which the agreement gate still does.
