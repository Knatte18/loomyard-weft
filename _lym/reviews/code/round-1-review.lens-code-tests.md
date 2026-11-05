```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens — for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. `settle` — an unproven liveness no longer settles.
   - Running turn stays busy: `TestSessionFold`, "an unproven liveness leaves a running turn busy" (`sessionstate_test.go`). Pinned.
   - Interrupt gives idle-stalled with cause interrupt: `TestSessionFold`, "...leaves an interrupted turn idle-stalled on the interrupt". Pinned.
   - Fresh session stays idle-fresh: `TestSessionFold`, "an unproven liveness leaves idle-fresh". Pinned.
   - An unreadable events file outranks the signals under unproven liveness: `TestSessionFold`, "an unreadable events file outranks the signals under an unproven liveness". Pinned.
   - A process-ending session end reads dead under unproven liveness: `TestSessionFold`, "a process-ending session end reads dead under an unproven liveness". Pinned.
   - Dead liveness outranks the signals: `TestSessionFold`, "a dead process outranks the signals, stamped with the reading". Pinned.
   - Empty signal history under unproven liveness reads no-signal: I believe the existing no-signal row covers it, because the zero `Liveness` is unproven. I did not re-read the row's facts, so this is unverified.
   - `SessionCauseLivenessUnproven` removed: the compiler pins it. Nothing needed.

2. Readiness and send follow the hook-derived state when the prober answers unproven.
   - `TestSend_WaitsForIdleSession`: the rows that lost `reading: true` now run on the default engine and read the hook state.
   - `strayRow` rows with `hook` run on `strayPasteEngine` without a `ProcessLiveness` override, so they pin it.
   - `TestRunner_SessionIdle_ReturnsScriptedClassification`: the unknown-reading rows now reach unknown through `unreadable`. The row "a hook turn end beside a pane needle reads idle and logs the disagreement" is unchanged.
   - Gate re-prompt rows hold on `activeFor` (operator active) instead of a busy pane. These pin the held send.
   - Test-fake changes: `tuiReed` gained `OperatorActive` and `working`.
   - The three deleted "under an unknown reading" Send rows had a different route to unknown, which now exists only via the unreadable file. `TestRunner_SessionIdle_ReturnsScriptedClassification` covers that route for readiness, but no Send-wait row reaches an unknown reading any more.

3. The wait loop no longer logs a disagreement for an unproven-liveness engine such as Codex.
   - Pinned by: none. `waitshadow_test.go` is unchanged, and no row has an unproven-liveness engine with a busy or idle-stalled state and asserts zero "session state disagrees" lines.

4. A Codex strand reads busy or idle-stalled through the real `readRunSessionState`.
   - Pinned by: none at that level. The fold-level rows prove `settle` only. No test drives `readRunSessionState` with an engine whose `ProcessLiveness` returns unproven and asserts the state.
   - `clearableBoxEngine` (`run_test.go:1795`) returns unproven but is a box-reading fake, not a state assertion.

5. `Spec.validate` refuses the hidden anchor.
   - `TestSpec_Validate`, "hidden anchor rejected, naming why and the own-window way forward". Pinned.
   - The own-window pass-through row is also updated.
   - "Before any strand exists" is not pinned by a Runner-level test, but `validate` is the first step in `run.go:559`, so the row is enough.

6. `lym shuttle run` refuses `--anchor hidden`.
   - `TestRunCLI_Run_FlagValidation`, "AnchorHidden". Pinned, asserting the full message.
   - "AnchorUnknown" and "overview" still assert only `own-window`. This is fine.

**Findings**

- **LOW, consistency** — `internal/battencli/wire_seams_test.go:415-421`. The fallback `wantState, wantCause = "unknown", "liveness-unproven"` can no longer be produced, because that cause is gone from the code. Where `proc.StartTime` is not readable, the assertion expects the removed cause and fails. Where it is readable, the registry-entry setup no longer decides the state, because an unproven liveness now yields idle-stalled/no-output as well. Remove the fallback and the registry setup, or assert `idle-stalled`/`no-output` unconditionally.
- **LOW, scope** — behaviour 3 and 4 above have no row. These are the two behaviours the task exists to fix at the system level: Codex state and the missing disagreement log. Add one row in `waitshadow_test.go`, or a `readRunSessionState` test with an unproven-prober engine, asserting a busy turn and zero disagreement warnings. The `settle` rows alone do not pin the wiring from `ProcessLiveness` to the fold.
- **NIT, scope** — the deleted Send-wait unknown-reading rows leave no `TestSend_WaitsForIdleSession` row that reaches an unknown reading. Whether the unknown fall-back-to-pane path in the send wait is pinned elsewhere is not shown in my context. Only add a row if no other test covers it.

**Tool calls:** none. The manifest context held every diff hunk and the callers I needed.
