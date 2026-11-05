---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/battencli/wire_seams_test.go:415
    origin: lens:code-affected-set
    summary: the test still expects the removed cause liveness-unproven as its fallback reading, which the fold can no longer produce
    fix: In TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker, delete the `wantState, wantCause = "unknown", "liveness-unproven"` default and the `if procStart, ok := proc.StartTime(...)` registry-entry setup with its comment, and set wantState, wantCause to "idle-stalled", "no-output" unconditionally, since an unproven liveness no longer changes the reading.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttlecli/run.go:122
    origin: lens:code-correctness
    summary: the --anchor refusal says reed starts no command for a hidden strand even for values that are not hidden, such as sideways or overview
    fix: In the `anchor != string(render.AnchorOwnWindow)` branch, build the message with the "reed starts no command for a hidden strand, so its provider never starts, and --focus=false keeps the viewer where it is" clause only when anchor == string(render.AnchorHidden), and otherwise return `invalid --anchor %q; want own-window`. Keep the AnchorHidden row in TestRunCLI_Run_FlagValidation asserting the full hidden text, and make the AnchorUnknown row assert the plain text.
---

### [LOW] Stale `liveness-unproven` expectation in the battencli activity test

**Location:** internal/battencli/wire_seams_test.go:415
**Issue:** `SessionCauseLivenessUnproven` is removed and an unproven liveness now settles nothing.
The test seeds `unknown/liveness-unproven` as the reading for a platform where `proc.StartTime` is unreadable.
That reading can no longer occur: the same run reads `idle-stalled/no-output` there, so the test fails on such a platform.
Where `proc.StartTime` is readable, the registry entry the test writes no longer decides the state.
The stale literal and the setup comment ("proven alive by this test process") describe the removed behavior.

### [LOW] The `--anchor` refusal blames hidden for every non-own-window value

**Location:** internal/shuttlecli/run.go:122
**Issue:** The guard now rejects every value except `own-window`, including `sideways` and `overview`.
Its message states "reed starts no command for a hidden strand, so its provider never starts" for all of them, which is wrong for a value that is not hidden.
The hidden refusal itself is correct: it comes before the prompt file is read and before `c.runner` runs, names why, and names own-window and `--focus=false` as the way forward.
The `AnchorUnknown` test row asserts only `own-window`, so it does not catch the misleading cause.

## Rejected

- Lens code-correctness F-B, an unproven Claude liveness (pid reuse, unreadable registry) behind a dead process keeping a stale busy reading: the amended entry says unproven liveness "only stops the state from claiming the process is alive; it does not replace the turn state", and the summary records the readiness change as intended. A dead process is still caught by `sessionGone` and by a proven-dead reading. Not a defect of the diff.
- Lens code-tests, no row pins the missing disagreement log or the real `readRunSessionState` wiring for an unproven-prober engine: the `TestSend_WaitsForIdleSession` rows that lost `reading: true` and the `strayRow` hook rows now run the real `readRunSessionState` over an engine whose default liveness is unproven and assert the hook-derived reading, and the `TestSessionFold` rows pin `settle`. The log's absence follows from `settle` returning the derived state, so a further row would duplicate that coverage.
- Lens code-tests NIT, no `TestSend_WaitsForIdleSession` row reaches an unknown reading after the three deletions: the summary explains that each deleted row duplicated its hook-reading neighbour, and `TestRunner_SessionIdle_ReturnsScriptedClassification` still reaches the unknown reading through an unreadable events file.

## Notes on the focus questions

- `settle` ranks the process-ending session end, a dead process, an unreadable events file, an empty signal history and then the derived state, as before, and the history rows in `TestSessionFold` pin it.
- The readiness, send and wait callers no longer read an unproven liveness as unknown, and none logs a disagreement for it.
- Hidden-anchor refusal: `Spec.validate` is reached only from `run.go:559`, before a run directory or strand exists.
  `Attach` does not validate and never starts a strand, and the hidden branches kept for old run records are reached only from a record.
  The stated cause matches reed: `planResumeLaunches` skips a hidden strand and `resolveLivePaneID` refuses to give it a pane.
  No contract, plugin skill or `docs/overview.md` text advertises `--anchor hidden` for shuttle runs.
- The diff changes no PATTERN entry, the directive declares none, and the commits carry no `Pattern-Departure:` trailer.
