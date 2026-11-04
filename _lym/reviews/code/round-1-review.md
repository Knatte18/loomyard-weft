---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:895
    summary: A held-turn-end redelivery goes through retryLanded, which reads the operator's turn start as the prompt having gone out, so the prompt is not typed yet DeliverPendingPrompt reports it delivered
    fix: In DeliverPendingPrompt (run.go), once it has decided the prompt's own turn never started and before the call to sendThroughPause, call clearPendingSend(run.runDir) and log a failure at Warn, so sendVerified's retryLanded finds no pending-send record for the prompt text and the line is really typed. Set promptDelivered only after the send typed it. Add a row to TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted whose fixture holds a pending-send record for the prompt line plus an operator turn start past its offset and asserts the line is typed, and a hold_test.go row for the same record at a held turn end.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/hold_test.go:306
    summary: deliverPromptAtHold's covered and turnStartRead guards and its send-failure branch, and deliverPromptOnAttach's send-failure branch, have no test row that fails when they break
    fix: In TestWait_HeldTurnEndNotice add prompt rows for a run covered by a pause (nothing typed), for a turn start read past the held turn end (nothing typed), and for sendFails (nothing delivered, the hold notice still sent, the next held turn end types the prompt). In TestAttach_DeliversTheDeferredPrompt add a row where the send fails and Attach and Wait still proceed. Fold these into the existing tables as rows; add no new test functions.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/hold.go:147
    summary: deliverPromptOnAttach has no covering-pause check where deliverPromptAtHold has one, so a covered attached run blocks Wait's entry inside the pause wait with no liveness or deadline polling
    fix: In deliverPromptOnAttach (hold.go) return early when the run is covered by a pause, as deliverPromptAtHold does with run.covered; this needs the pause state observed before the call, so call run.observePause() first in Wait at wait.go:295 or leave the attach delivery to the first held turn end. Add the matching row from F2.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/command.go:195
    summary: A failure after the session directory is created leaves it behind half-written, and Open removes it only when AddStrand fails
    fix: In writeOpenFiles (command.go) remove absDir with os.RemoveAll on every error return after MkdirAll created it, so OpenLaunch leaves nothing on a failure. Add a TestRunner_Open row where the fake engine's OpenLaunch creates the directory and AddStrand fails, asserting open.go:501 removed it.
  - id: F5
    severity: LOW
    class: scope
    origin: handler
    location: internal/fabricengine/recordswritelock_test.go:23
    summary: The commit 8a813147 changes a fabricengine lock test, which the Goal's two shuttle faults do not license, and the change description does not mention it
    fix: Revert the hunks of internal/fabricengine/recordswritelock_test.go from this branch so the diff holds only the shuttle change; if the flaky wait must be fixed, defer it to the run's parent to land on its own.
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/attach_test.go:94
    summary: Two TestAttach_DeliversTheDeferredPrompt rows (operator turn start, prompt's own turn start) restate the matching rows of TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted
    fix: Keep the zero-offset, set-offset and output-exists rows in TestAttach_DeliversTheDeferredPrompt, which pin the Wait-entry wiring, and drop its operator-turn and own-turn-start rows, which TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted already pins.
---

### [BLOCKING] Held-turn-end redelivery can type nothing and still count as delivered

**Location:** internal/shuttleengine/run.go:895, send.go:632, run.go:1593
**Issue:** This is the Part 1 scenario of the Goal, and the new instruction match does not reach it.
When `loadSkillsThenPrompt`'s send fails with `ErrSubmissionNotLanded`, `pasteVerified` (send.go:576) writes a pending-send record for the prompt text with the pre-send events offset.
The operator then types, which writes a turn start past that offset.
At the held turn end `deliverPromptAtHold` calls `DeliverPendingPrompt`, which sees no turn start carrying the prompt and calls `sendThroughPause`, and that calls `sendVerified`.
`sendVerified` calls `retryLanded` (send.go:648), which returns true when the record's text equals the text and any turn start lies past the offset, with the operator's turn start qualifying.
The send returns nil having typed nothing, `DeliverPendingPrompt` sets `promptDelivered` and reports true, the hold notice is suppressed, and no later held turn end retries.
The agent still has no prompt.
The unchanged late-landing acceptance at run.go:838, which takes any turn start as the prompt landed, is the reason Start returns the run in this state, and the held-turn repair is the only thing the diff relies on to cover it.
The new tests use `echoReed` with no pending-send record, so none exercises this path.

### [MEDIUM] Guards and failure branches of the new delivery have no test row

**Location:** internal/shuttleengine/hold_test.go:306, internal/shuttleengine/attach_test.go:94
**Issue:** The diff adds two success rows to `TestWait_HeldTurnEndNotice`.
Deleting the `run.covered` guard, which would type into a paused pane, or the `run.turnStartRead` guard, or the send-failure fall-through to the hold notice, fails no test; the `sendFails` field exists in the table and no prompt row sets it.
The failure branch of `deliverPromptOnAttach` is likewise untested.

### [LOW] deliverPromptOnAttach does not check for a covering pause

**Location:** internal/shuttleengine/hold.go:147
**Issue:** `deliverPromptAtHold` returns early on `run.covered`, but `deliverPromptOnAttach` does not.
For a covered attached run `DeliverPendingPrompt` enters `sendThroughPause`, which holds `sendMu` and blocks in `waitWhileCovered` until the pause lifts, and `Wait` runs no liveness or deadline polling meanwhile.

### [LOW] A failed open leaves a half-written session directory

**Location:** internal/shuttleengine/claudeengine/command.go:195
**Issue:** `writeOpenFiles` creates the directory and writes `settings.json`; if the append-prompt write then fails, `OpenLaunch` returns the error and `Open` (open.go:79) returns without removing the directory, because only the `AddStrand` failure path removes it (open.go:501, uncovered by a test).
Nothing removes the directory or its sibling `<token>.exited` marker after a session ends; the marker behaved so before this diff, and the diff adds files to the same leak.

### [LOW] Unlicensed lock test change

**Location:** internal/fabricengine/recordswritelock_test.go:23
**Issue:** The Goal asks for two shuttle launch fixes.
The third commit changes the wait a fabricengine test gives a free lock, which neither fix needs, and `_lym/landing/summary.md` does not mention it.
Docs for the changed behaviour (`docs/overview.md`, `creelengine/doc.go`, `shuttleengine/doc.go`, `claudeengine/doc.go`) do land in the same diff and are licensed by the Goal.

### [NIT] Attach test rows restate the matching rows of the run test

**Location:** internal/shuttleengine/attach_test.go:94
**Issue:** The operator-turn and own-turn-start rows of `TestAttach_DeliversTheDeferredPrompt` pin matching logic that `TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted` already pins.

## Rejected

- A lock-ordering test for `DeliverPendingPrompt` taking `sendMu` before the record read (code-tests NIT): the move is internal, and the public behaviour is pinned by the once-per-handle assertion.
- A `/`-leading prompt line never matching `promptTurnStartedSince` (code-correctness): `typedInstruction` drops a leading-slash turn start by design, and the registered prompt pointer lines start with a word, so no real path reaches it.
- An engine without `SessionSignalParser` retyping a delivered prompt (code-correctness): both registered engines implement the parser, and the summary names the reliance as a bound.
- Part 2 placement (handler): the settings go to a unique per-token file named by `--settings`, never into the worktree's own settings, so a second open of a role cannot overwrite or merge with an earlier one, and an open outside a hub (empty board directory) writes nothing and is unchanged.
- Run path (handler): `buildSettings` still builds the hook settings once, through `creelFlagDirForHook` and `creelWaitHookEntry`, and the run launch is unchanged.
- Change description (handler): its Part 1 and Part 2 claims match the diff, apart from the omission recorded under the unlicensed lock test finding.
