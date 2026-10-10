---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-landing-pre-merge-clear.md
    summary: conflictSessionStopper matches only the exact role conflict and stops one strand, so a live numbered conflict-2 session is missed and the abort runs under it, breaking D2's stop-before-abort bound
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-publish-verify-failed-fix-route.md
    summary: the recorded-failure Stuck reason keeps today's trailing fix forward on the task branch, then resume and appends the goto, so it names two ways forward, one of them the hand fix D1 exists to replace
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/07-decision-add-refuses-task-session.md
    summary: the parent-recorded refusal's way forward omits that the operator's own unnamed shell may always run the verb, which D6 and its open risk require in both variants
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-landing-pre-merge-clear.md
    summary: cards 5 and 6 never state in their Intent that the abort and the narrowed guard also discard or let through a parent-sourced merge-in an operator or the orch started by hand, with its partial resolution, nor what bounds that
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/07-decision-add-refuses-task-session.md
    summary: card 7's Intent does not state the new check's bound, that it reads an unverified environment variable and so guards a mistaken call, not an agent that clears or forges it
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/plan/01-midmerge-record-fields.md
    summary: SourceSHA and StartSHA are each one field, but the record holds a warp and a weft value of each, and the card never says which side fills them or what the test asserts against
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/06-resume-start-own-leftover.md
    summary: the new recordedParentBranch seam set in newLoomCLI is left out of TestNewLoomCLI_SetsInjectedSeams, the test that pins every seam newLoomCLI sets, so a nil production seam goes uncaught
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/08-stopped-young-run-dir-rows.md
    summary: the new ProbeGated sub-case repeats the Untracked stopped row through the same attach body, while the burler half-probe row D5 names is replaced by an argument
---

### [MEDIUM] The conflict-session stopper misses a numbered conflict strand

**Location:** `_lyx/plan/05-landing-pre-merge-clear.md`, the `conflictSessionStopper` paragraph.

**Issue:** The card has the stopper find "the live, non-retiring strand whose name parses through `agentname.Parse` to the role `mergeresolve.ConflictRole`" and stop that one strand.
Reed numbers a role that is already held: `internal/reedengine/strand.go:206-217` collects the held roles of every tracked strand, live or dead, and forms the name with `agentname.NumberRole(role, held)`, which returns `conflict-2`, `conflict-3` and so on (`internal/agentname/agentname.go:115-128`).
`internal/mergeresolve/spec.go:107` spawns the conflict session with `Role: conflictRole` and no fixed name, so it goes through that numbering.
The resolver runs up to `maxConflictAttempts = 2` sessions per call, and a dead `conflict` row still in the table pushes the next session to `conflict-2`.
A driver that dies during that second session leaves a live `conflict-2`.
The stopper finds no strand with the exact role `conflict`, returns `("", nil)`, and `clear` aborts the merge while that session keeps writing the conflicted paths.
D2 requires stopping "any live conflict-session strand of its own run", and its Bound says "no session the row failed to stop is left writing into a redone merge".
The table in `TestConflictSessionStopper` has no row for a numbered strand or for two live conflict strands, so the gap would pass review.

**Fix:** Have the stopper match the role `ConflictRole` and its numbered forms (`ConflictRole` or `ConflictRole-<N>`), stop every live, non-retiring match, and return the guid of the first stop that fails beside its error.
Add rows to `TestConflictSessionStopper` for a live `conflict-2` strand and for two live conflict strands.

### [MEDIUM] The recorded-failure Stuck reason names two ways forward

**Location:** `_lyx/plan/03-publish-verify-failed-fix-route.md`, the `verdict` sentence.

**Issue:** The card says `verdict` "keeps today's reason and, when `recorded` is true, ends it with the way forward" for the goto.
Today's failed reason at `internal/landingshed/verifygate.go:216` already ends with `; fix forward on the task branch, then resume`.
Appending the goto clause therefore gives a reason that first tells the hub to fix the task branch by hand and resume, then tells it to run the Webster-Burler goto.
D1 exists because "the hub keeps choosing between a plan re-run and a hand fix", and it says the reason "ends with the way forward": run the goto, then resume.
The unchanged text should stay only where `recorded` is false, which `TestVerifyGate_CheckMapsVerifyResult` pins (`verifygate_test.go:197`).

**Fix:** When `recorded` is true, replace the `fix forward on the task branch, then resume` clause with the goto way forward rather than appending to it.
Keep today's full text when `recorded` is false.
In `TestPublishVerify_FailureRecord`, assert that the plan-verify and `publish_verify` rows' reason does not contain `fix forward on the task branch`.

### [MEDIUM] The parent-recorded refusal omits the operator's unnamed shell

**Location:** `_lyx/plan/07-decision-add-refuses-task-session.md`, the `decisionVerb` sentence that gives the two way-forward clauses.

**Issue:** With a parent recorded, the card's way forward is "a design call after the Discussion is the parent's to record; ask <parent name> to run the verb".
Only the no-parent variant mentions the operator's own unnamed shell.
D6 says the way forward "names the run's recorded parent when one is recorded, else the operator, as the caller who runs the verb, and says the operator's own unnamed shell may always run it".
The decision record's first open risk depends on that clause: for a pair created from a task worktree, the named parent `<shortname>:<slug>:orch` is held by no session until `orch-any-worktree` lands, "the operator's unnamed shell still passes, and the refusal says so".
As planned, a refusal in that case points only at a parent nobody can reach.
The `TestDecisionVerb_Refusals` row for "a named caller differing from the recorded parent" would pin the incomplete text.

**Fix:** Have both refusal variants end with the statement that the operator's own unnamed shell may always run the verb, and have the parent-recorded row in `TestDecisionVerb_Refusals` assert it.

### [MEDIUM] The abort's reach is unbounded in cards 5 and 6

**Location:** `_lyx/plan/05-landing-pre-merge-clear.md` and `_lyx/plan/06-resume-start-own-leftover.md`, the `**Intent:**` of each.

**Issue:** Card 5 calls a parked `merge-in` whose `Source` is `parentBranch` "the row's own leftover" and aborts it.
Card 6 lets `resume` and `start` through over the same state.
Neither Intent says what that predicate also covers: a `lyx fabric merge-in` of the parent that an operator or the orch started by hand in the pair, which the abort discards with any partial resolution.
Neither Intent states the bounds either.
D2's Bound gives them: only a conflict-session strand of the run's own pair is stopped, the stop runs only when the row is about to abort, the abort runs only after a successful stop, and another verb or source and all foreign state are never touched.
D3's Bound gives the guard's: whatever D2's abort discards it lets through, a session that outlived a dead driver is stopped by the row before the abort, and every other row keeps the refusal.
The rubric's attack-surface rule requires the card that bypasses a guard or adds a discard to state what it can now let through and what bounds it.
The cards state the mechanics, but a reviewer of card 5 or card 6 alone cannot tell that a hand-started merge-in is in scope.

**Fix:** In card 5's Intent, add a sentence that the abort discards any parked parent-sourced `merge-in` at Publish or Finalize entry, whoever started it, a hand-started one with its partial resolution included, followed by D2's bounds.
In card 6's Intent, add a sentence that the exception lets through exactly what card 5's abort discards, followed by D3's bound.

### [LOW] The decision-add caller check states no bound

**Location:** `_lyx/plan/07-decision-add-refuses-task-session.md`, `**Intent:**`.

**Issue:** The card lists what passes: an empty strand name, the recorded parent's name, and the operator's unnamed shell.
It never says that the check rests on `LYX_STRAND_NAME`, which a session can clear or set.
The check therefore guards a mistaken call, not a determined agent.
D6's Bound states this, and also that a future driver route (`run-message-routing`) will need its own label or exemption.
The empty-name pass is the check's escape hatch, and the rubric's attack-surface rule wants that limit stated on the card.

**Fix:** Add D6's Bound to the Intent: the check reads only the exported name and the recorded parent name, so it guards a mistaken call, not one that clears or forges the variable, and no driver exemption is given now.

### [LOW] MidMergeState's two SHAs do not say which side they come from

**Location:** `_lyx/plan/01-midmerge-record-fields.md`, `**Intent:**`.

**Issue:** `mergeState` (`internal/fabricengine/mergestate.go:47-61`) records `WarpStart`/`WeftStart` and `WarpSource`/`WeftSource`.
Its doc comment says that for a merge-in the weft is recorded as unmoved, and `WeftSource` is "legitimately empty when no counterpart resolves".
The card adds one `SourceSHA` and one `StartSHA` and describes them as "the resolved SHA it merged and the pre-merge HEAD SHA it recorded", without saying which side.
It also asserts them as "the pre-merge HEAD and the source branch's SHA" without naming the repository.
An implementer who fills them from the weft would log an empty or unmoved SHA in card 5's abort line, and the test would be written against whichever side was picked.

**Fix:** State that `SourceSHA` and `StartSHA` are filled from the record's code-side (`WarpSource`, `WarpStart`) values, and have the integration test assert them against the task worktree's code HEAD before the merge and the parent code branch's SHA.

### [LOW] newLoomCLI's new seam is outside the seam test

**Location:** `_lyx/plan/06-resume-start-own-leftover.md`, the `recordedParentBranch` sentence.

**Issue:** Card 6 adds a `recordedParentBranch` field to `loomCLI` and sets it in `newLoomCLI`.
Every planned test row substitutes a canned branch.
`TestNewLoomCLI_SetsInjectedSeams` (`internal/loomcli/cli_test.go:110-125`) is the test that pins newLoomCLI's seams, `midMerge` among them, because no other test sees the production receiver.
The card does not list `internal/loomcli/cli_test.go`, so a production `newLoomCLI` that leaves the seam nil would pass every test and panic on the first parked merge.

**Fix:** Add `internal/loomcli/cli_test.go` to card 6's `**Edit:**`, and have `TestNewLoomCLI_SetsInjectedSeams` assert that the seam is non-nil, updating its `testtiming:keep` line, which names the fields it pins.

### [LOW] Card 8's ProbeGated sub-case duplicates an Attach row

**Location:** `_lyx/plan/08-stopped-young-run-dir-rows.md`, `**Intent:**`.

**Issue:** `Attach` and `ProbeGated` both run the one probe body `attach` (`internal/shuttleengine/attach.go:89-92`).
The new `stopped_young_record_is_not_found` sub-case seeds an untracked strand with a `stopped` record and a young directory and expects not-found.
That is the same disposition the card's own `stopped` row in `TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge`'s Untracked sub-table pins, so `PATTERN-test-economy` counts it as a redundant case.
D5 names rows in the attach table and in the burler half-probe table.
The card adds the first, replaces the second with a sub-case on the shuttle side, and then argues that no burler row is needed.

**Fix:** Drop the `ProbeGated` sub-case.
Either add a `stopped` row to the burler half-probe table D5 names, or state in the Intent that `TestEngine_ProbeRound`'s existing not-found rows (gone, or done with output) are the burler coverage D5 asks for, since the burler probe sees only shuttle's not-found answer.

## Focus verdicts

- Fidelity, cards 3 and 7 against D1 and D6: each deliverable is carried by a card. Card 3 carries the orch escalation item and the `Publish: verify failed` row. Card 7 carries the `decision: task session` row, the dropped no-caller-identity sentence (file header and group `Long`), the `docs/overview.md` `decision add` line, and the plan stencil text including the no-parent case. The gaps are the wording faults in F2 and F3.
- Fidelity, card 2 against D4: the three-point comparison, the parent tip in the notice, and all four scenarios are carried. The task-edit and unsynced-parent scenarios are the existing rows `a listed config file changed is reported` and `a commit on the parent after the fork does not appear` in `TestReadConfigChanges`, and the synced and identical scenarios are new rows. Not a defect.
- Fidelity, card 3 against D1: the timeout, could-not-start, dirty and failed-write reasons keep naming no goto, and `recordFailure`'s answer reaches `verdict`. Not a defect beyond F2.
- Granularity, card 5: `preMerge`, `newPreMerge` and `preMerge.clear` are one unit on one card. Three `Create:` handles are what the format requires for three new members, not a split. `conflictSessionStopper` belongs on card 5: `TestLandingDeps_EveryFieldPopulated` fails on any unfilled `Deps` field, so the seam and its production wiring cannot land on separate cards. Not a defect.
- Test economy, cards 5 and 6: card 6 extends `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals`, `..._CleanPairProceeds` and `TestResumeVerb` rather than adding functions. The pre-merge step and `conflictSessionStopper` are new behavior that no existing test covers, so `premerge_test.go` and `TestConflictSessionStopper` are not redundant. Not a defect.
- Member glyphs and re-sign arrows, cards 3, 5 and 6: `recordFailure`, `verdict`, `landingDeps` and `refuseOverUnfinishedMerge` carry arrows. `check`, `checkPublishVerify`, `NewPublish`, `NewFinalize`, `runDriverSpawnAndWait` and `resumeFromRunState` keep their signatures. No `Edit` card names symbols only through a Go file path. Not a defect.
- Attack-surface bound, cards 5, 6 and 7: see F4 and F5.
- Card order, cards 1, 4, 5 and 6: cards 5 and 6 use `MidMergeState` fields from card 1 and `ConflictRole` from card 4, both earlier cards. `conflictRole` has one use, `spec.go:107`, which card 4 covers. Not a defect.
- Verify coverage: `## verify:` runs `./...` through build, vet under `integration`, `tmux` and `llm`, `go test` and `go test -tags integration`, in the template's order. The only nested module is test data. Not a defect.
