```yaml
lens: code-correctness
outcome: reported
```

Task: review the code-correctness lens, running the diff's changed functions and branches on concrete inputs.

## Per function or branch

1. **`pasteVerified` / `clearBoxBeforeCommand`**
   - Case: a command send where the operator types between the idle read and the clear. Not correct (finding 1).
   - Case: the clear key fails. Correct: it returns before the paste, `sentAt` is untaken and no pending record is saved.
   - Case: the engine has no `InputBoxClearer`. Correct: it returns nil and pastes as before.
   - Case: a retry after a failed clear. Correct: the clear replays on each `pasteVerified` call.
2. **`confirmPasted` with `startsNoTurn`**
   - Case: a turn start past `sentAt` that the command did not cause. Not correct (finding 2).
   - Case: a box that reads empty and a landing-start line, with no turn start. Correct: unchanged behaviour.
   - Case: the window closes with no landing. Correct: it still returns `ErrSubmissionNotLanded` through `failUnlanded`.
3. **`retryLanded`**
   - Case: a pending record for a command text. Correct: the `!sc.startsNoTurn` guard, which the diff did not touch, makes it read as not landed and clears the record.
   - Case: `ErrCommandTakenAsPrompt` saves no pending record. Correct: `pasteVerified` saves one only for `ErrSentTextInDraft` or `ErrSubmissionNotLanded`.
4. **`mapDeliveryError`** (`delivery.go:462`)
   - Case: `ErrCommandTakenAsPrompt`. It falls to `default` and passes through unchanged. In `typeInject` (`deliver.go:425-430`) that reports the send error, requeues the inject and returns false. The requeue is as the summary says, but it has no bound (finding 3).
5. **`OpenLaunch` TTL export**
   - Case: POSIX, mapped role `orch`. Correct: `export CLAUDE_CODE_PROMPT_CACHE_TTL='5m'; &lt;resume&gt; || &lt;fresh&gt;`. The export covers both halves of the `||`, and `Runner.Open` chains the marker and self-remove after it.
   - Case: pwsh. Correct: `$env:K = '5m'; &amp; claude … || &amp; claude …`. The `||` chain stays intact and the variable reaches the child.
   - Case: unmapped role. Correct: it resolves the default TTL.
   - Case: an invalid TTL anywhere in `claude.yaml`. Correct: it refuses before `writeOpenFiles`, and `Runner.Open` has made no strand by then.
   - Case: an empty role. Correct: `Runner.Open` refuses it first. At the engine level it takes the default, as a run does.
6. **`sendSequence` for an engine without `TextPaster`, `InputBoxReader` or `SessionCycler`.** Correct: unchanged, with no clear and no taken-as-prompt check.

## Findings

1. **MEDIUM, design:** the clear can wipe operator text typed after the idle read.
   - Location: `send.go:623-627`, with the guards in `awaitIdleSession` and `busyReading` (`send.go:282`).
   - The activity hold and the empty-box read happen in `sendVerified`, before `pausedRefusal` and `retryLanded`, which reads and rewrites the pending-send file. `clearBoxBeforeCommand` then sends `C-u` with no fresh activity check or box read.
   - Operator text typed in that gap is deleted silently. Before the diff, the paste merged with it and failed as `ErrSentTextInDraft` or `ErrForeignBoxText`, so the draft survived. `guardedEnter` already checks operator activity immediately before its key, and the clear has no such check.
   - The summary's "goes only into a box the guard has just read as empty, after the operator-activity hold" overstates the guarantee.
   - Evidence: `send.go:623-627, 641-654, 796-803`; `run.go:1551-1565`.
2. **MEDIUM, design:** any turn start past the paste fails the command, so a command that ran is requeued and typed twice.
   - Location: `send.go:754-757`.
   - `turnStartedSince` accepts any turn-start signal after `sentAt`: an operator prompt submitted during the window, a hook, or a subagent or queued turn. It does not match the command text, unlike `promptTurnStartedSince`, which matches by instruction.
   - If `/compact`, `/clear` or `/color` landed and an unrelated turn started in the same window, the send now returns `ErrCommandTakenAsPrompt`. Creel requeues the inject and types the command again, so the compact or clear runs twice.
   - The check also runs before the landing-start check at line 762, so a session-start line that proves the command ran does not override it.
   - Evidence: `send.go:754-763, 951-953`.
3. **MEDIUM, design:** the requeue after `ErrCommandTakenAsPrompt` has no bound, and each retry delivers the command to the agent as a prompt again.
   - Location: `delivery.go:460-476`, with `deliver.go:425-431`.
   - If the engine's clear does not remove the leading text, such as whitespace that `C-u` does not reach or a box with several whitespace rows, every retry pastes the command behind it. Each retry starts a prompt turn and fails again, and the worker requeues each time without limit or backoff.
   - The agent receives ` /compact …` as an instruction on every cycle. The summary's "types it again once the session is idle" holds only if the clear works, and nothing handles the case where it does not.
   - The `ErrNotLanded` path was avoided for this error on purpose, but no retry cap replaces it.
   - Evidence: `deliver.go:425-431`; `delivery.go:462-476`; `claudeengine/session.go:227-229`.

## Tool calls
None. The inherited context already held every definition I needed.
