---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/20-turn-end-counts-with-nothing-outstanding.md
    summary: "card 20 removes the transcript-shell expiry but leaves recoverbatch_test.go out of Edit, so its expiry row fails on the card's own gate"
    origin: lens:test-gaps
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: "the no gate arrival yet test read from gateFails, gateSent and gatePending misses a failing arrival whose re-prompt send failed or was deferred"
    origin: lens:correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: "finalize is told to log that the strand removal ended the shells, but finalize removes a strand only on a Done end without KeepPane, so deadline and liveness ends log a removal that did not happen"
    origin: lens:error-handling
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/24-recovery-retry-on-the-verbs.md
    summary: "card 24 rewords the Master dead rungs that claims_test.go pins, but that file is not in Edit"
    origin: lens:generic
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-run-auto-rebaseline.md
    summary: "finishMasterDone must receive the rebaseline warning as its own input but is listed without the re-sign arrow and the Intent never says how the warning reaches it"
    origin: lens:generic
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-fabric-reference-policy-at-record-batch.md
    summary: "no planned test pins webstercli supplying IsReadOnlyCommand or auditTerminalFork and runExitAuditCrossCheck passing it through, so dropping the classifier there leaves every row green"
    origin: lens:test-gaps
  - id: F7
    severity: MEDIUM
    class: scope
    location: _lyx/plan/13-read-only-classifier-allow-list.md
    summary: "TestIsReadOnlyCommand gets no row for the new || separator and none for the lyx fabric readers list, pairs, status, shortname, prune, cleanup or the rejected prune --apply, cleanup --apply, shortname x"
    origin: lens:test-gaps
  - id: F8
    severity: MEDIUM
    class: scope
    location: _lyx/plan/19-fabric-way-forward-names-the-orch.md
    summary: "the reworded resetCmd unknown-parent refusal has no test and reset_test.go is not listed, and the substring-matched texts have no assertion pinning the new clause"
    origin: lens:test-gaps
  - id: F9
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-rebaseline-refusals-keyed.md
    summary: "the rows asserting the captured Info log live in parallel tables, but log capture swaps the process-global logger, so the card must make them non-parallel under a comment naming it"
    origin: lens:test-gaps
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/plan/03-rebaseline-refusals-keyed.md
    summary: "no step-set row reaches the overview-read transient, and the claim that an empty Step keeps today's texts contradicts wrapping the restampBaseline failure"
    origin: lens:error-handling
  - id: F11
    severity: LOW
    class: scope
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: "Run.Wait still reads held.tasks and noticeMaxTasks in hold.go becomes unused, yet neither is in Edit"
    origin: lens:correctness
  - id: F12
    severity: LOW
    class: design
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: "expiredTurnEnd clears run.waitingTasks before an at-once Done, so finalize would record no shells unless the card says the list is kept"
    origin: lens:correctness
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: "the card says the wait marker shows from the first shell-only turn end and also only past ShellWaitBound, and says Warn where logPayloadShellPastBound logs at Info"
    origin: lens:generic
  - id: F14
    severity: LOW
    class: scope
    location: _lyx/plan/21-one-shell-wait-rule-and-ended-shells.md
    summary: "the background_shell_wait_min comment in shuttleengine template.yaml and a waitshadow_test.go row name go stale but neither is listed"
    origin: lens:test-gaps
  - id: F15
    severity: LOW
    class: consistency
    location: _lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md
    summary: "warnings the in-line route records still read accepted by accept-audit --batch, because only the refusal's re-run verb is parameterised"
    origin: lens:correctness
  - id: F16
    severity: LOW
    class: scope
    location: _lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md
    summary: "no row covers a mixed batch of read-only fabric references plus another uncheckable kind, or a fabric reference that carries a path, both of which must still refuse needs_fresh"
    origin: lens:test-gaps
  - id: F17
    severity: LOW
    class: design
    location: _lyx/plan/11-pattern-entry-line-cap.md
    summary: "importing internal/pattern gives planparser a transitive tree of stencil, stencilstore, gitexec, lyxcwd, logger and proc that the card neither states nor reconciles with doc.go's leaf wording"
    origin: lens:generic
  - id: F18
    severity: LOW
    class: scope
    location: _lyx/plan/11-pattern-entry-line-cap.md
    summary: "TestValidate_PatternEntryLineCap has no row for a fence with an info string, an unclosed fence or two fenced blocks in one card"
    origin: lens:test-gaps
  - id: F19
    severity: LOW
    class: scope
    location: _lyx/plan/07-plan-gate-done-card-edited.md
    summary: "neither card 7 nor card 11 adds a row to internal/loomcli/parity_test.go although recent gate checks each got one there"
    origin: lens:generic
  - id: F20
    severity: LOW
    class: design
    location: _lyx/plan/08-webster-record-note.md
    summary: "an unreadable webster record renders none, telling the reviewer nothing is frozen while card 7's gate errors on the same record, and no row pins that path"
    origin: lens:error-handling
  - id: F21
    severity: LOW
    class: consistency
    location: _lyx/plan/06-edited-done-cards.md
    summary: "TestEditedDoneCards repeats the edited done card row that card 7's TestNewPlanGate already covers through ValidatePlan"
    origin: lens:generic
  - id: F22
    severity: LOW
    class: scope
    location: _lyx/plan/07-plan-gate-done-card-edited.md
    summary: "no row covers an id the plan lacks in card 6 or the unreadable card file propagating as an error from ValidatePlan in card 7"
    origin: lens:test-gaps
  - id: F23
    severity: LOW
    class: consistency
    location: _lyx/plan/19-fabric-way-forward-names-the-orch.md
    summary: "the card says no test reads ReadOriginFor's missing-lock text, but hubgeom's geometry_integration_test.go already asserts it, so the new top-level TestReadOriginFor and its budget bump are avoidable"
    origin: lens:test-gaps
  - id: F24
    severity: LOW
    class: consistency
    location: _lyx/plan/24-recovery-retry-on-the-verbs.md
    summary: "resetCmd's Long help says reset changes no other webster state, which the batch-start count clear falsifies, and the card does not say the help changes"
    origin: lens:generic
  - id: F25
    severity: LOW
    class: design
    location: _lyx/plan/13-read-only-classifier-allow-list.md
    summary: "FabricReaderVerbs is an exported declaration read across packages by card 17 yet is bundled into the classifier card instead of getting its own card"
    origin: lens:generic
  - id: F26
    severity: LOW
    class: scope
    location: _lyx/plan/16-gate-and-verify-drop-the-strand-name.md
    summary: "runTest changes three environment branches but one TestGateTest_Scenario row is planned and does not say which branch it drives"
    origin: lens:test-gaps
  - id: F27
    severity: LOW
    class: consistency
    location: _lyx/plan/17-fabric-sandboxed-role-guard.md
    summary: "the fabriccli budget rises by one only if roleguard_test.go holds exactly one top-level test, and the planned lyx config row does not fit the hub-wide guardRows shape"
    origin: lens:test-gaps
  - id: F28
    severity: LOW
    class: scope
    location: _lyx/plan/04-run-auto-rebaseline.md
    summary: "no row covers Fresh set beside AutoRebaseline, a returned error after an accepted rebaseline opening with the warning, or the stated Warn logs of cards 4 and 23"
    origin: lens:test-gaps
  - id: F29
    severity: LOW
    class: scope
    location: _lyx/plan/04-run-auto-rebaseline.md
    summary: "after an accepted rebaseline the plan-validation refusal still returns a plain error naming lyx webster rebaseline, which the adapter turns into a hard failure instead of Stuck"
    origin: lens:error-handling
  - id: F30
    severity: LOW
    class: scope
    location: _lyx/plan/20-turn-end-counts-with-nothing-outstanding.md
    summary: "the acceptance row that a recovery with a shell outstanding and its report present classifies from the report has no planned test row"
    origin: lens:test-gaps
  - id: F31
    severity: NIT
    class: scope
    location: _lyx/plan/01-gitexec-run-stdin.md
    summary: "TestRun's table has no stdin field or stdout check so the planned RunStdin rows need a structural change the card does not state"
    origin: lens:test-gaps
  - id: F32
    severity: NIT
    class: scope
    location: _lyx/plan/22-round-gate-reruns-the-top-level-test.md
    summary: "the collapse keeps first-appearance order per package but the only row has two subtests of one test in one package"
    origin: lens:test-gaps
---

### [BLOCKING] Card 20 leaves a test that asserts the removed expiry out of Edit

**Location:** `_lyx/plan/20-turn-end-counts-with-nothing-outstanding.md`
**Issue:** `TestRecoverBatch_SecondCall` in `internal/websterengine/recoverbatch_test.go` (line 674) has the row "a turn end waiting on a transcript-reported shell is dead once background_shell_wait_min has passed".
Card 20 makes `TurnEndedAfter` count a turn end only with nothing outstanding, so that row fails on the card's own gate.
The plan template's rule on reworded behavior puts every test asserting it on the card that rewords it, and the card lists only `strand_test.go`.
The new top-level `TestTurnEndedAfter_OutstandingWork` also repeats two rows `TestRecoverBatch_SecondCall` already pins (payload shell outstanding, plain turn end), which `PATTERN-test-economy` counts as redundant.
**Fix:** Add `internal/websterengine/recoverbatch_test.go` to `Edit:`, flip the transcript-shell row to the still-running verdict and refresh the test's doc comment.
Keep the new test only for cases the existing surface cannot reach.

### [MEDIUM] The "no gate arrival yet" test has a hole

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`
**Issue:** The card reads "the gate has evaluated no arrival of this run yet" from `gateFails`, `gateSent` and `gatePending`.
`handleGatedBoundary` can evaluate a failing arrival and leave all three at zero or false: the re-prompt send fails busy or unlanded (`unsentReprompt` set, memo cleared, neither count moved), or `boundaryIdle()` defers an arrival that already landed.
`gateFails` also resets on a pass, and `gateSent` is never reset.
A later waiting turn end with the earlier arrival's leftover files and an unawaited shell then counts at once, which is the case D9 says must wait.
**Fix:** Add a dedicated per-run flag set by every real `evaluateGate` (or by the first gated Done arrival) and have the at-once rule read it.
Name `unsentReprompt` as covered and add a row for the failed-send case.

### [MEDIUM] The ended-shells Info line is logged where no removal happens

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`
**Issue:** The card has `finalize` log at Info that the strand removal ended the shells.
In `internal/shuttleengine/wait.go` `finalize` removes a strand only when `outcome == OutcomeDone && !KeepPane`, and a failed `RemoveStrand` warns and continues.
A deadline or liveness end (`OutcomeTimeout`), a `KeepPane` run and a failed removal would all log a removal that did not occur, and the card's test rows ask for the line on exactly the deadline and liveness ends.
**Fix:** Log inside the cleanup branch beside a successful `RemoveStrand`.
For non-Done ends log that the run ended with shells outstanding and leave removal to the caller, then align the test rows.

### [MEDIUM] Card 24 rewords stencil rungs that `claims_test.go` pins

**Location:** `_lyx/plan/24-recovery-retry-on-the-verbs.md`
**Issue:** `contracts/stencils/claims_test.go` pins `"OR \`status: dead\`"` (line 490) and `` "`dead` → its recovery already failed" `` (line 496) for `webster-template-master.md`.
Card 24 splits the stuck-or-dead rung and rewords the entry-time dead rung, and lists only `cli_test.go`.
The plan template requires every test asserting a reworded text to join the card.
**Fix:** Add `//contracts/stencils/claims_test.go` to `Edit:`, state which strings the claims keep or replace, and add a claim for the `recovery_retry` and `recovery_exhausted` rungs.

### [MEDIUM] `finishMasterDone` is re-signed in effect but not in the plan

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`
**Issue:** `finishMasterDone` takes `freshWarnings []string` and nothing that identifies the rebaseline warning (`runlevel.go:755`).
The card has it write a `Plan rebaselined` section and open a stuck reason with the warning, so it needs the warning as its own input.
The `Edit:` bullet has no re-sign arrow and the `ImpactSummary` names no changed caller.
**Fix:** Write `internal/websterengine#finishMasterDone -> func finishMasterDone(...)` with the added parameter and state the caller change in `ImpactSummary`.

### [MEDIUM] Card 14 pins the classifier at no CLI or loom surface

**Location:** `_lyx/plan/14-fabric-reference-policy-at-record-batch.md`
**Issue:** The rows are `ClassifyViolation` cases, `RecordBatch` with an injected classifier and a loomcli wiring assertion.
Nothing pins `auditTerminalFork` or `runExitAuditCrossCheck` passing `ReadOnly` through, or `webstercli`'s `runDeps` and `recordBatchCmd` supplying `fabricengine.IsReadOnlyCommand`.
The acceptance row "a fork transcript with only read-only fabric references records-batch done" is unreached at the CLI.
**Fix:** Add a `webstercli` row driving `record-batch` with a read-only reference and a `Run` exit-cross-check row, and list those files in `Edit:`.

### [MEDIUM] Card 13 leaves the new separator and the `lyx fabric` readers unpinned

**Location:** `_lyx/plan/13-read-only-classifier-allow-list.md`
**Issue:** D8 says the rewrite adds rows for `||`, yet the card lists none, and today's table has none.
The card gives one accepted row per `git` form but none per `lyx fabric` reader (`list`, `pairs`, `status`, `shortname`, `prune`, `cleanup`) and no rejected rows for `prune --apply`, `cleanup --apply` and `shortname x`.
An over-rejecting or over-admitting `readOnlyLyxSegment` passes.
**Fix:** Add `ls a || cat b` accepted and `ls a || git push` rejected, one accepted row per reader and the three rejected forms.

### [MEDIUM] Card 19 rewords a refusal no test reaches

**Location:** `_lyx/plan/19-fabric-way-forward-names-the-orch.md`
**Issue:** `resetCmd`'s "the parent branch is unknown … run `lyx fabric reconcile`" text (`internal/webstercli/reset.go:143`) gets the clause, and no test contains that text; `reset_test.go` is not listed.
`PATTERN-refusal-way-forward` requires a reworded webster refusal to be reached by a test.
For the texts the card calls substring-matched, no planned assertion pins the new clause, so dropping it from `pushFailureWayForward`, `decisionVerb` or `validateNaming` stays green.
**Fix:** Add `internal/webstercli/reset_test.go` with a row forcing the unknown-parent refusal and asserting the clause, and add the clause to one assertion per substring-matched text.

### [MEDIUM] Card 3's log-asserting rows meet a process-global logger

**Location:** `_lyx/plan/03-rebaseline-refusals-keyed.md`
**Issue:** `TestRebaseline_OverviewFrameUnreadable` and `TestRebaseline_EditedPlan` are parallel tables, and `logcapture.Capture` swaps the logger's output and verbosity for the process.
`PATTERN-test-speed` drops `t.Parallel` for a test touching global state and names that state in a comment; the card says neither.
**Fix:** State that the three log-asserting rows run non-parallel under a comment naming the logger.

### [LOW] Card 3 leaves one transient unreached and contradicts itself on empty Step

**Location:** `_lyx/plan/03-rebaseline-refusals-keyed.md`
**Issue:** The card keys "each transient retry" but its rows cover the changed-file read and the card-hash read only; the `overviewIndexOnly` / `recordedOverviewFrame` read failure has no step-set row.
It also says an empty `Step` keeps every text as today's, yet wraps the `restampBaseline` failure, which is returned bare today.
**Fix:** Add the overview-read row, and either exempt the restamp wrap from the claim or add an empty-step restamp row.

### [LOW] Card 21 omits `Run.Wait` and the orphaned notice bound

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`
**Issue:** `Run.Wait` (`wait.go:270`) logs `len(held.tasks)`, and the card drops `heldTurnEnd.tasks`.
`noticeMaxTasks` in `hold.go` is unused once `holdNotice` drops its listing.
**Fix:** Add `internal/shuttleengine#Run.Wait` to `Edit:` and state that `noticeMaxTasks` goes.

### [LOW] `expiredTurnEnd` clears the waiting list `finalize` is to read

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`
**Issue:** `expiredTurnEnd` sets `run.waitingTasks = nil` before returning Done (`wait.go:821`).
The card fills `EndedShells` in `finalize` from the waiting list, so the gated at-once finish would record no shells.
**Fix:** Say in the Intent that `waitingTasks` is kept until `finalize` reads it.

### [LOW] Card 21 contradicts itself on the marker and the log level

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`
**Issue:** `shellWaitActive` is on "from the first such turn end", while `ShellWaitBound` is the threshold after which the shell is "shown in the wait marker", and the `waitmark_test.go` rows assert the marker "past the bound".
The card also says "logged once at Warn through `logPayloadShellPastBound`", which logs at Info today (`wait.go:838`).
**Fix:** Pick one marker rule and make the rows match, and say the level moves to Warn or drop it from the sentence.

### [LOW] Card 21 leaves stale comments and a stale row name

**Location:** `_lyx/plan/21-one-shell-wait-rule-and-ended-shells.md`
**Issue:** The `background_shell_wait_min` comment in `internal/shuttleengine/template.yaml` says such shells never expire and counts as a turn end; D9 makes it a log and marker threshold.
The `waitshadow_test.go` row "after transcript-reported shells expired" goes stale too.
**Fix:** Add `template.yaml` to `Edit:` and rename the row.

### [LOW] In-line accepted warnings name the wrong verb

**Location:** `_lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md`
**Issue:** `AcceptBatchFabricReference` records `Detail: "accepted by accept-audit --batch, " + basis` (`auditledger.go:248`); card 15 parameterises only the refusal's re-run verb, so warnings recorded through `recover-batch` name `accept-audit` in `summary.md`.
**Fix:** Word the recorded detail from the same verb parameter.

### [LOW] Card 15 has no mixed-batch or path-carrying rows

**Location:** `_lyx/plan/15-recover-batch-accepts-read-only-fabric-references.md`
**Issue:** The in-line route needs every `Uncheckable` entry to be a pathless read-only fabric reference, and the rows cover only all-read-only and a non-read-only entry.
**Fix:** Add a mixed batch row and a fabric reference with a path row, each asserting `needs_fresh`.

### [LOW] Card 11 widens `planparser`'s dependency tree without saying so

**Location:** `_lyx/plan/11-pattern-entry-line-cap.md`
**Issue:** `planparser` imports `lyxdirs` and quarry `glyph` only; `internal/pattern` brings `stencil`, `stencilstore`, `gitexec`, `proc`, `lyxcwd` and `logger` transitively.
`planparser/doc.go` calls the package a pure leaf that never imports `lyxcwd`, and the card edits `doc.go` without addressing it.
The record's notes name the cap's source, so the cap is licensed and the unstated edge is the gap; no cycle results.
**Fix:** State the new import in the Intent and reword the `doc.go` sentence, or inject the cap as `FabricReferenceMatcher` is.

### [LOW] Card 11 covers one fence shape

**Location:** `_lyx/plan/11-pattern-entry-line-cap.md`
**Issue:** "Only fenced blocks are scanned", but no row covers a fence with an info string, an unclosed fence or two fenced blocks in one card.
**Fix:** Add those rows to `TestValidate_PatternEntryLineCap`.

### [LOW] No parity row for the two new gate checks

**Location:** `_lyx/plan/07-plan-gate-done-card-edited.md`
**Issue:** The constraint binds `PATTERN-gate-self-check-parity` for D2 and D13, and the recent gate checks each got a `wantFinding` row in `internal/loomcli/parity_test.go`; cards 7 and 11 add none.
**Fix:** Card 11 adds a `pattern-entry-line-cap` row, card 7 adds a `done-card-edited` row if its record fixture is cheap, and both list the file in `Edit:`.

### [LOW] An unreadable record renders `none` in the rubric

**Location:** `_lyx/plan/08-webster-record-note.md`
**Issue:** The reviewer and fixer are told nothing is frozen while card 7's gate returns an error on the same record, and no row pins the unreadable path.
**Fix:** Render a note that the record could not be read and done cards are to be treated as frozen, keep the Warn and add the row.

### [LOW] `TestEditedDoneCards` repeats a row card 7 covers

**Location:** `_lyx/plan/06-edited-done-cards.md`
**Issue:** The card says the test pins what card 7's rows do not, then lists "an edited done card reported", which card 7's `TestNewPlanGate` row already covers through `ValidatePlan`.
**Fix:** Drop that row and keep the unreadable-file error, the begun-not-done case and the nil state.

### [LOW] Card 6 and card 7 error paths have no row

**Location:** `_lyx/plan/07-plan-gate-done-card-edited.md`
**Issue:** Card 6 says an id the plan lacks contributes nothing and card 7 says an unreadable card file is a returned error from `ValidatePlan`; neither has a row.
**Fix:** Add one row to each.

### [LOW] Card 19's premise about `ReadOriginFor` is false

**Location:** `_lyx/plan/19-fabric-way-forward-names-the-orch.md`
**Issue:** The card says no test reads `ReadOriginFor`'s missing-lock text, but `internal/hubgeom/geometry_integration_test.go:157` asserts the `ResolveParent` error names the lock directory and `lyx fabric reconcile`.
The new top-level `TestReadOriginFor` and its budget bump are avoidable.
**Fix:** Pin the clause in the hubgeom row, list that file, and drop the new test or keep it as a unit-level addition.

### [LOW] `lyx webster reset` help goes stale

**Location:** `_lyx/plan/24-recovery-retry-on-the-verbs.md`
**Issue:** `resetCmd`'s `Long` says the reset changes no other webster state apart from the `--to start` archive; the batch-start count clear falsifies it.
**Fix:** Add an Intent sentence that the help names the count clear.

### [LOW] `FabricReaderVerbs` is bundled into the classifier card

**Location:** `_lyx/plan/13-read-only-classifier-allow-list.md`
**Issue:** It is an exported declaration used across packages by card 17, which the rubric's granularity rule gives its own card.
**Fix:** Split it into a small card before card 13 that cards 13 and 17 `Uses:`, or state in the Intent why it stays.

### [LOW] Card 16 plans one row for three branches

**Location:** `_lyx/plan/16-gate-and-verify-drop-the-strand-name.md`
**Issue:** `runTest` builds its environment in the slotted, inherited-slot and unslotted branches, and `Verify`'s changing case is the nil-env one; one `TestGateTest_Scenario` row does not say which branch it drives.
**Fix:** Name the branch the row drives or add rows for the others.

### [LOW] Card 17's budget count and config row are loose

**Location:** `_lyx/plan/17-fabric-sandboxed-role-guard.md`
**Issue:** "Rises by one" holds only if `roleguard_test.go` holds exactly one top-level test, which the card never says.
The `TestConfigOverRealHub` `guardRows` table is shaped for hub-wide writes, and the per-worktree subtest uses an injected sync, so the planned row does not show `lyx config`'s own route passing.
**Fix:** State the top-level test count and place the row in the per-worktree subtest, driving the real sync wiring if the claim stays.

### [LOW] Card 4 lacks three rows its Intent promises

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`
**Issue:** No row covers `Fresh` set beside `AutoRebaseline` ("keeps its archive path"), a returned error after an accepted rebaseline opening with the warning while matching `ErrPendingAuditFindings`, or the Warn logs stated in cards 4 and 23.
**Fix:** Add the three rows, the log ones non-parallel as in the card 3 finding.

### [LOW] A validation refusal after an accepted rebaseline stays a hard error

**Location:** `_lyx/plan/04-run-auto-rebaseline.md`
**Issue:** After an accepted rebaseline, Run's plan-validation refusal still returns a plain error whose text says to run `lyx webster rebaseline --card NN` then `lyx webster run`, so the adapter fails the row with `producer-hard-failure`.
A Plan-loop edit that introduces a validation defect is a likely trigger.
D1's acceptance covers rebaseline refusals only, so this is a gap and not a violation.
**Fix:** Key that text on `deps.reentryStep()` and wrap it in `ErrAutoRebaseline`, or state in the Intent that validation refusals stay hard errors.

### [LOW] Card 20 has no report-present row

**Location:** `_lyx/plan/20-turn-end-counts-with-nothing-outstanding.md`
**Issue:** The acceptance row "a recovery strand's turn end with a shell outstanding does not count; with its report present the batch classifies from the report" has no row for the second half.
**Fix:** Add a row with a shell outstanding and the report present.

### [NIT] Card 1's table needs a structural change it does not state

**Location:** `_lyx/plan/01-gitexec-run-stdin.md`
**Issue:** `TestRun` calls `gitexec.Run(tt.args, tt.dir)` with no stdin field and no stdout check, and `TestRun_StdoutOnError` already covers the stdout-kept case for `Run`.
**Fix:** Say the table gains a `stdin` field and a `RunStdin` dispatch, and put the stdout-kept case beside `TestRun_StdoutOnError`.

### [NIT] Card 22's row does not exercise ordering

**Location:** `_lyx/plan/22-round-gate-reruns-the-top-level-test.md`
**Issue:** The collapse keeps first-appearance order per package, but the only row has two subtests of one test in one package.
**Fix:** Add an interleaved row (`TestA/s1`, `TestB`, `TestA/s2`, and one name in two packages).

## Rejected

- Card 24's `card_amended` re-run row as a duplicate of card 23's row (test-gaps): it pins the verb's envelope at the CLI surface, a different public surface than the engine row.
- Rubric-excluded items were not raised: dependency-field completeness, `ImpactSummary` on non-Edit cards, and anything the plan gate already checks.
