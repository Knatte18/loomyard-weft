---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/startup_test.go:1146
    summary: TestStartup_SendFailuresTearDown and the rewritten TestRun_Send_ConfirmsSubmission neither call t.Parallel nor name the global state that would excuse it (PATTERN-test-speed)
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/wait.go:388
    summary: maxUnsentReprompts ends the gate loop after five consecutive unlanded re-prompts, a bound the plan and decision D6 do not carry (they bound the retry by the run deadline only)
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/send.go:1
    summary: send.go file header describes only the send context and the idle wait, but the file also holds the settle, confirm and clear path and the gate-loop methods boundaryIdle, startHoldReleased and sendWithin
---

### [BLOCKING] Two tests are serial with no named reason

**Location:** `internal/shuttleengine/startup_test.go:1146` (`TestStartup_SendFailuresTearDown`) and `internal/shuttleengine/run_test.go:1066` (`TestRun_Send_ConfirmsSubmission`).

**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless the test touches process-global state (env, cwd or a shared fixture), and requires a comment at the test or once atop its file to name that state.
Neither test calls `t.Parallel`, neither parent nor subtests, and neither carries such a comment.
`TestRun_Send_ConfirmsSubmission` was serial before this diff only because each row replaced the package-level `inputSleep`, and its old doc line said so.
The rewrite onto a `fakeClock` removed the `stubInputSleep` use and also removed that sentence, so the remaining serial run has no reason.
`TestStartup_SendFailuresTearDown` has no `stubInputSleep`, no `logcapture` and no shared fixture: each row builds its own `newFixture` over `t.TempDir()`.
Both tests are the new shuttle send path's main tables, so they set the serial cost for the package.
Per the constraints section, a listed-constraint violation is BLOCKING however small.

**Fix:** Add `t.Parallel()` to both tests and their `t.Run` bodies, or, if a row does touch global state, name it in a comment at the test.

### [LOW] An unsent re-prompt count cap that the plan does not carry

**Location:** `internal/shuttleengine/wait.go:131` (`maxUnsentReprompts`) and `internal/shuttleengine/wait.go:388`.

**Issue:** Card 6's Bound says the re-prompt retry spends no gate budget and is bounded by the run deadline, and decision D6 says only "any other re-prompt send error still ends the loop".
The diff also ends the loop with `finalize(OutcomeDone)` on the fifth consecutive busy or unlanded re-prompt at one boundary.
Its stated reason is a clock that never advances, which only a test clock does; in production each failed send already takes at least one tick, so the cap's only live effect is to end a gate early, after roughly five idle waits, where the plan keeps retrying until the deadline.
The same shape of cap in `awaitIdleSession`, `awaitSettledBox` and `confirmSubmitted` (`windowPollCap`) is sized past what the real windows allow, so it never binds in production; this one does.
The behaviour is documented in `wait.go` and pinned by `TestGate_UnsentRepromptRetries`, so the departure is deliberate but undeclared against the plan.

**Fix:** Either drop the cap and let the test's clock advance on `Sleep`, or keep it and have the decision record state it as part of D6 so the diff and the fasit agree.

### [LOW] send.go file header is narrower than the file

**Location:** `internal/shuttleengine/send.go:1`.

**Issue:** The header says the file holds what a verified send does before it types: the context and the idle wait.
The file also holds `awaitSettledBox`, `confirmSubmitted`, `clearUnlandedText`, the `ErrSubmissionNotLanded` sentinel, and `Run.sendWithin`, `Run.boundaryIdle` and `Run.startHoldReleased`, which belong to the gated wait loop.
A reader choosing a file by its header would not look here for the submit window or the clear, and would not expect the gate-boundary methods.

**Fix:** Widen the header to name the settle, confirm and clear path and the gate-loop helpers, or move the three `Run` gate methods to `wait.go` beside `handleGatedBoundary`.

## Focus verdicts

- **Card 4, clear and `ErrSubmissionNotLanded`:** no defect.
  `clearUnlandedText` plays the clear only for an engine with both `SessionCycler` and `InputBoxClearer`, and only when the box is copies of the normalized sent text (the last possibly a needle-long prefix) or a `PastePlaceholder`; a foreign draft gets no key and is named in the error.
  `ErrSubmissionNotLanded` surfaces from the unsettled box, the closed window after an Enter or at the Enter count, the never-appeared text, and a window closed before typing.
  A late landing (empty box before the clear) returns nil, as the plan decides, including after a never-appeared text; the test table pins that row.
- **Cards 3 and 4, send context and clock:** no defect.
  The idle polls, appearance polls, settle reads and confirm backoff all sleep through `sc.clock`; the remaining `inputSleep` uses are `playInputs`' per-step `SettleMS` and the existing agent-pane probe, both left by the decision.
  `deadline` is set only in `Run.sendWithin`, and `newSendContext` leaves it zero.
- **Card 5, start teardown:** no defect.
  Every failure past `AddStrand` in `awaitStartup` (the three status-cap exits included) and `loadSkillsThenPrompt` (loader, load turn, offset persist, prompt delivery) goes through `abandonStartup(OutcomeDied, cause)`, wrapping `ErrNotStarted` beside the cause; the late-landed prompt starts the run.
- **Card 6, gate boundary idle:** no defect beyond F2.
  A busy or unlanded re-prompt leaves the failure and sent counts unchanged, clears the memo, sets the unsent count and is re-entered by `Wait`'s per-tick branch; any other send error, and a failed attempt that reaches `gateFails`, still counts.
- **Cards 7 to 10, stop verb:** no defect.
  Burler's `stopHalf` and `stopLiveHalf` use `Handle.Stop`; webster's entry, begin-batch, recover-spawn, awaitTerminal and reset reclaims use `StrandStopper`; attach's superseded removal uses `stopRecorded`.
  A repository grep finds no remaining caller of `StrandRemover`, `NewReedStrandRemover`, `RemoveStrandIfLive`, `removeStrandIfLive` (webster) or `BurlerRemover`, and `remover.go` with its test is gone; the remaining raw `RemoveStrand` callers are shuttle's own teardowns, reed, orch and the loom driver, which `PATTERN-shuttle-stop` excepts.
- **Test economy:** no finding.
  The new top-level tests each cover a behavior no existing test covers (`stop_test.go` forms, `TestClearInputSequence`, `TestPastePlaceholder`, `TestRemoveRecoveryStrands` on the stopper); `TypeSequence` is asserted inside `TestComposeSend` rather than by a new test, and the status-cap rows were corrected in place.

Verification run: `go build ./...`, `go vet` under the `integration` and `tmux` tags, `go test` over the touched packages and `cmd/lyx`, and `go test -race ./internal/shuttleengine/` all pass.
