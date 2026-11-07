---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/send.go:623
    summary: The clear before a command paste sends C-u with no fresh activity check or box read, so operator text typed after the idle read is wiped silently.
    fix: In clearBoxBeforeCommand (send.go:645), before playInputs, call operatorActivityHold(sc.reed, sc.guid, sc.activityFailures) as guardedEnter does at send.go:797 and return a *SessionBusyError wrapping ErrSessionBusy when it holds, as awaitIdleSession does at send.go:196. Also read the box through readInputBox and send the clear only when it reads as blank or unreadable; a box with text returns an error wrapping ErrForeignBoxText and sends no key. Add a row to checkPastedSessionCommand for each hold. The change description's claim that the clear goes only into a box the guard has just read as empty should then be true as written.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/send.go:754
    summary: confirmPasted fails a command with ErrCommandTakenAsPrompt on any turn start past the paste, so a command that ran beside an unrelated turn start is requeued and typed twice.
    fix: In confirmPasted, for a send with startsNoTurn, replace the turnStartedSince test with promptTurnStartedSince(sc.engine, sc.eventsPath, sentAt, text), which matches a turn start whose instruction equals the trimmed pasted text, and return ErrCommandTakenAsPrompt only on that match. Any other turn start past the paste then falls through to the landing-start and box checks like any other signal. Add a row to checkPastedSessionCommand where an unrelated turn start follows the paste and the command is confirmed.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelcli/delivery_test.go:461
    summary: No test pins that mapDeliveryError passes ErrCommandTakenAsPrompt through, the one step that makes creel requeue the inject instead of journalling it.
    fix: Add one row to the mapDeliveryError table in internal/creelcli/delivery_test.go near line 461, with fmt.Errorf wrapping shuttleengine.ErrCommandTakenAsPrompt as input and the same wrapped error expected out, so a new case mapping it to ErrNotLanded or ErrBusy fails. Add a row to the existing table, not a new test function.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/send.go:645
    summary: Two branches of clearBoxBeforeCommand have no test, a failed clear key that must paste nothing and an engine without InputBoxClearer that must still paste.
    fix: Add two rows to checkPastedSessionCommand in internal/shuttleengine/session_test.go. One has the fake reed fail SendKey and expects an error and no PasteSubmit call. The other drives a paste-path engine that lacks InputBoxClearer, which parserlessPasteEngine no longer is since the diff added the capability to it, and expects one paste and no key.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelengine/doc.go:412
    summary: The command inject contract in creelengine and creelcli docs names only ErrBusy and ErrNotLanded, though a command taken as a prompt is now a third outcome that is requeued and retyped.
    fix: In internal/creelengine/doc.go near line 412 and internal/creelcli/doc.go near lines 149-150, add one sentence each stating that a command the provider took as a prompt fails as a plain send error, so creel requeues the inject and types it again once the session is idle.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:676
    summary: The overview says the engine sets the prompt-cache TTL on launch and resume, but sessions opened with lym shuttle open now get it too.
    fix: In docs/overview.md line 676, change on launch and resume to on launch, resume and open.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/template.yaml:3
    summary: The prompt_cache_ttl and prompt_cache_ttl_roles comments say the values are validated when a run starts, but OpenLaunch now validates them and refuses an open too.
    fix: In internal/shuttleengine/claudeengine/template.yaml lines 3 and 4, change validated when a run starts to validated when a run starts or a session opens.
---

### [MEDIUM] The clear can wipe operator text typed after the idle read

**Location:** internal/shuttleengine/send.go:623
**Issue:** The operator-activity hold and the empty-box reading both happen in `sendVerified` through `awaitIdleSession`, ahead of `pausedRefusal` and the pending-send file read in `retryLanded`.
`clearBoxBeforeCommand` then sends `C-u` with no second activity check and no box read.
Text the operator types in that gap is deleted without trace.
Before this diff the paste merged with such text and failed as `ErrSentTextInDraft` or `ErrForeignBoxText`, so the draft survived.
`guardedEnter` at send.go:797 shows the repository's own habit of checking activity immediately before a key.
The change description's claim that the clear goes only into a box the guard has just read as empty holds at read time, not at key time.

### [MEDIUM] Any turn start past the paste fails the command, so a command that ran can be typed twice

**Location:** internal/shuttleengine/send.go:754
**Issue:** `turnStartedSince` accepts any turn-start signal past `sentAt`: an operator prompt submitted in the window, a hook, or a queued or subagent turn.
For a send that starts no turn, `confirmPasted` now turns that into `ErrCommandTakenAsPrompt`, and it does so before it reads the landing-start line at send.go:762 that would prove the command ran.
Creel's `typeInject` default branch (deliver.go:425) then requeues the inject, so a `/clear` or `/compact` that ran is typed again.
A `/clear` repeated after the operator started new work wipes that work's context.
`promptTurnStartedSince` (send.go:958) already matches a turn by its instruction, and the taken-as-prompt turn is exactly one whose instruction equals the trimmed pasted text.

### [LOW] No test pins the pass-through of ErrCommandTakenAsPrompt in mapDeliveryError

**Location:** internal/creelcli/delivery_test.go:461
**Issue:** The diff changes only a comment in `mapDeliveryError`, and the requeue it describes rests on the error reaching the `default` branch.
The tables at delivery_test.go:407-409 and :461 hold no row for the error, and no creel test sets it, so a case mapping it to `ErrNotLanded` or `ErrBusy` would pass every test.

### [LOW] Two branches of clearBoxBeforeCommand are untested

**Location:** internal/shuttleengine/send.go:645
**Issue:** No row injects a failing `SendKey`, so the return before the paste ("nothing was pasted") is never reached.
The diff added `InputBoxClearer` to `parserlessPasteEngine`, so no paste-path engine without the capability remains and the skip branch is never reached either.
Dropping the `ok` check or pasting after a failed clear would pass every test.

### [LOW] Creel docs list two outcomes for a command inject, the diff adds a third

**Location:** internal/creelengine/doc.go:412
**Issue:** `creelengine/doc.go:412` says a command inject has the same `ErrBusy` and `ErrNotLanded` contract as a typed line, and `creelcli/doc.go:149-150` lists the same two answers.
A command taken as a prompt now ends as a plain send error that creel requeues, and only the `mapDeliveryError` comment says so.

### [LOW] The overview describes the TTL as set on launch and resume only

**Location:** docs/overview.md:676
**Issue:** Line 676 says the engine sets the prompt-cache TTL per role on launch and resume.
`OpenLaunch` now exports it for opened sessions too, so the sentence leaves the open path out.

### [NIT] The template comments name run start as the only validation point

**Location:** internal/shuttleengine/claudeengine/template.yaml:3
**Issue:** Both TTL entries say the values are validated when a run starts.
`OpenLaunch` (command.go:454) now validates them and an invalid entry refuses the open.

## Rejected

- Unbounded requeue after `ErrCommandTakenAsPrompt` (code-correctness finding 3, MEDIUM): the failed paste submits the whitespace-led text as a prompt, which empties the box, so the next attempt starts from a clean box; a loop needs a source that retypes the whitespace each time, which nothing in the evidence shows, and requeue without a cap is how `typeInject` already treats every other send error.
- Pending-send state not asserted after `ErrCommandTakenAsPrompt` (code-tests finding 4, NIT): `pasteVerified` saves a record only for `ErrSentTextInDraft` and `ErrSubmissionNotLanded`, and `retryLanded` never reads one as landed for a command, so the behaviour follows from untouched code.
- Invalid-TTL row covers only the default, not a bad role entry (code-tests finding 5, NIT): the resolver is shared with `Prepare` and tested through it, so a second row would repeat that coverage.

## Focus verdicts

- `Runner.Open` has one caller: true, `internal/shuttlecli/open.go:90` is the only construction of an `OpenSpec` and the only call of `Runner.Open`.
- An empty `spec.Role` cannot reach `OpenLaunch` through `Runner.Open`, which refuses it at open.go:65; an unmapped role resolves to the default TTL, as a run's does, through the shared `resolvePromptCacheTTL`.
- `sh.Chain(exportTTL, ...)` is correct in both dialects: POSIX `export K='5m'; resume || fresh` and pwsh `$env:K = '5m'; & claude ... || & claude ...` cover both halves, and `Runner.Open` chains the marker and self-remove after the line.
- Part 1 of the Goal, finding where the space came from: the change description states the source is unknown and the diff claims no cause, which matches the record.
- Test economy: the new rows each pin a behaviour no existing test covers; no duplicate found.
