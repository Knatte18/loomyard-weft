---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shuttleengine/send.go:312
    summary: Retry loops around real spawns (the confirm Enter loop, the unsent gate re-prompt resend) are capped by elapsed time only, which PATTERN-spawn-observability forbids; the old sendExtraEnters count cap was deleted
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/websterengine/runlevel.go:287
    summary: Webster's reclaim sites lost the Warn context the deleted removeStrandIfLive carried, although card 9 says to keep it in the caller's log line
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/template.yaml:7
    summary: Config and template comments misdescribe the send keys (submit window opens at typing not after an Enter, slash commands are not verified sends, submit_settle_ms still paces the Enter in the command sequences)
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/run.go:1135
    summary: A send whose window closed before anything was typed reports that the text never appeared after 2 attempts and that the TUI swallowed it
  - id: F5
    severity: NIT
    class: consistency
    location: internal/shuttleengine/wait.go:6
    summary: File header still calls abandonStartup the not-ready teardown after card 5 widened it to every start failure past AddStrand
---

### [BLOCKING] Count-uncapped retry loops around real spawns

**Location:** `internal/shuttleengine/send.go:312` (`confirmSubmitted`), `internal/shuttleengine/send.go:290` (`awaitSettledBox`), `internal/shuttleengine/wait.go:384` (`handleGatedBoundary`'s unsent re-prompt, re-entered from `Wait`).

**Issue:** `PATTERN-spawn-observability` says "A retry loop around a real spawn caps attempt COUNT, not only elapsed time."
Its stated reason (the startup tick cap) is that the cap also ends the loop under a clock that never advances.
Before this diff the Enter retry was capped by `sendExtraEnters = 2`.
Card 4 deletes it ("`sendExtraEnters` and its fixed count go").
`confirmSubmitted` now sends an Enter (a real `tmux send-keys` through reed) and re-reads the box in a `for` loop that ends only on confirmation or `windowClosed(sc.clock, closeAt)`.
`awaitSettledBox` re-reads the box (a real `capture-pane`) until two reads agree or the same window closes.
The gate's unsent re-prompt is re-sent on every later tick until it lands or `run.deadline` passes, and each resend is a full type, Enter and capture cycle; it has no count either.
All three terminate only because the injected `Clock.Sleep` advances `Now`.
With the real clock, `submit_settle_ms: 0` (a legal value) makes `awaitSettledBox` read back to back for as long as the box keeps changing.
Cards 4 and 6 prescribe the window-only bound ("the window caps how long and therefore how many").
The constraint conflicts with that plan text, and the constraint wins; this finding is satisfiable in the diff alone.

**Fix:** Give each loop an attempt-count bound next to the time bound: a named maximum Enter count in `confirmSubmitted` (sized so the doubling schedule still reaches the cap interval), a maximum read count in `awaitSettledBox`, and a maximum number of unsent re-prompt resends per boundary in `handleGatedBoundary`/`Wait`, after which the loop falls back to today's "re-prompt send failed, ending the loop" path.
Add a row or assertion to the existing `TestRun_Send_ConfirmsSubmission` and `TestGate_UnsentRepromptRetries` pinning the cap under a clock that does not advance.

### [MEDIUM] Webster's stop sites dropped the operator-facing Warn

**Location:** `internal/websterengine/runlevel.go:287` (`reclaimEntryTimeStrands`), `internal/websterengine/beginbatch.go:392`, `internal/websterengine/recoverbatch.go:189` and `:586`.

**Issue:** Card 9 says each site keeps "its error wrapping and its Warn context in the caller's log line where the old helper carried it".
The deleted `removeStrandIfLive` logged `websterengine: stopping a leftover live strand before respawning it; an in-flight batch run restarts from state.json rather than resuming`, a Warn written so an operator of the handback-row `Webster` step can see that a reinvocation killed the in-flight Master.
None of the four callers logs anything now (no `logger` call in `runlevel.go`'s reclaim, `beginbatch.go` or `recoverbatch.go` around the stops).
The only trace left is shuttle's generic `shuttle: stopping a live strand`, which carries a guid and none of the restart-from-state.json cost.

**Fix:** Log the Warn with its original wording and the guid at each of the four webster call sites (or once in a small webster helper they share) before `StopStrand`, as card 9 requires.

### [LOW] Send config comments disagree with the code

**Location:** `internal/shuttleengine/template.yaml:7-10`, `internal/shuttleengine/config.go:30-36`.

**Issue:** Three drifts in the comments card 1 rewrote.
`submit_confirm_timeout_s` is described as "seconds after an Enter within which the send must be seen to land", but `sendVerified` opens the window when typing begins (`closeAt` is set before `TypeSequence` plays), as `doc.go` says.
`send_ready_timeout_s` lists "the /clear, /compact and /model commands" as verified sends that wait for an idle session; the only `sendVerified` callers are prompt and skill-load sends, `Run.Send`, `Runner.Send` and `Runner.LoadSkills`, and the command sequences are played with `playInputs`.
`submit_settle_ms` is now described only as the interval between the two agreeing box reads, but `claudeengine` still uses it as the pause before the Enter in `ClearSessionSequence`, `CompactSessionSequence`, `ReloadPluginsSequence` and `ModelSwitchSequence`.

**Fix:** Reword the three comments to match: window from the moment typing begins, drop the slash commands from `send_ready_timeout_s`, and keep the command-sequence role in `submit_settle_ms`'s comment.

### [LOW] "Never appeared" error when nothing was typed

**Location:** `internal/shuttleengine/run.go:1135`.

**Issue:** When the submit window (cut to the run deadline) is already closed before the first `playInputs`, the loop breaks with `typed == false` and the send fails with "sent text never appeared in the pane after 2 attempt(s) — the provider TUI likely swallowed the input; the send was NOT delivered".
Nothing was typed, so the message names a cause and an attempt count that did not happen, and it points the operator at the TUI instead of at the exhausted deadline.
A gate re-prompt started near the run deadline reaches this path.

**Fix:** Give the `!typed` case its own message (the window had closed before typing began), still wrapping `ErrSubmissionNotLanded`, and extend the existing deadline row of `TestGate_UnsentRepromptRetries` or `TestRun_Send_DeliveryVerification` to assert it.

### [NIT] Stale file header in wait.go

**Location:** `internal/shuttleengine/wait.go:6`.

**Issue:** The header says `abandonStartup` is awaitStartup's "not-ready teardown".
Card 5 widened it to the teardown of every start failure past `AddStrand`, and its own doc comment and `ErrNotStarted`'s were updated, but this header was not.

**Fix:** Reword the header to match `abandonStartup`'s doc comment.

## Focus verdicts

- Send order and clock: `sendVerified` runs `awaitIdleSession`, types with `TypeSequence`, runs the appearance check, then `awaitSettledBox`, `confirmSubmitted` and `clearUnlandedText`, in the order cards 3 and 4 name; every timed wait goes through `sc.clock` (only `playInputs`' per-step `SettleMS` stays on `inputSleep`, as the Shared Decision allows). Not a defect.
- Unlanded sends: the clear runs only for an engine with `SessionCycler` and `InputBoxClearer`, only into a box matching `isOwnText` or `PastePlaceholder`, and a foreign draft gets no key. A short prefix below the needle length stays in the box by design. Not a defect beyond F4's message.
- Cards 5 and 6: a start-time send or persist failure goes through `abandonStartup(OutcomeDied, cause)` and removes the strand; an unlanded or busy gate re-prompt spends no attempt and cannot end the loop. The only gap is F1's missing count bound.
- Cards 7 to 10: `stopped` is recorded before removal in `Run.Stop`, `Runner.StopStrand` and attach's superseded removal. No production caller outside shuttle removes a shuttle run's strand (the remaining `RemoveStrand` callers are reed's, orch's and the loom driver and status strands). The grep for `StrandRemover`, `NewReedStrandRemover`, `RemoveStrandIfLive` and `removeStrandIfLive` finds nothing outside shuttle's own `removeStrandIfLive` and a generated benchmark report.
- Delete and Rename groups: card 10's Delete group has zero remaining references, and card 4's `sendExtraEnters` has none; the build, `go vet` under the integration, tmux and llm tags, and the race-enabled tests of `shuttleengine`, `burlerengine` and `websterengine` pass.
- Test economy: no duplicate test found. The new tests each cover behavior no existing test covers, and `test-budget.yaml` matches the added test functions.
- Docs: `doc.go` of shuttleengine and websterengine, `PATTERN-shuttle-stop` and the `PATTERN-completion-signal` edit match the behavior; the config findings are F3.
