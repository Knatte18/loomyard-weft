---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/waitshadow.go:132
    summary: the wait loop's fact refresh passes only the tick's new signals, so the transcript interrupt and API-error markers are read only at the tick a turn start or turn end arrives and never re-read at later liveness ticks
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/signals.go:3
    summary: two comments cite plan artifacts that do not outlive the task, "card 3's parse" in smoke_turns_test.go:378 and "the provider-seam containment decision" here, instead of a stable identifier
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/session_test.go:292
    summary: TestRunner_SessionState neither calls t.Parallel nor names the global state that would keep it serial, against PATTERN-test-speed
---

Range: no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`), 18 commits, one per card.

What I ran: `go build ./...`, `go vet` untagged and under `integration`, `tmux` and `llm`, `go test ./...`, `go test -tags integration ./...` and `lyx loom lint-comments --base <merge-base>`.
All passed.
The `llm`-tier scenario (`TestSmokeTurnsSessionHooks`) was compiled by the vet step and not run.

Focus questions, each settled:

- Shadow-only sites (13, 14, 15): every state read in `logSessionState`, `logIdleStateDisagreement` and `readSessionStatesByStrand` only logs.
  A failed read returns before any decision, and the tests compare the `Result`, notifier calls, saved `State` and queue with the read failing against the read succeeding.
  Not a defect.
- Card 4 before card 5: commit `04cad24e9` (card 4) precedes `f35bc845b` (card 5).
  `eventsFileActivity` counts only the newest `Event` signal, so an `idle_prompt` append, a stamp line or any other skipped line never moves `LastActivity`.
  Not a defect.
- Provider seam: `shuttleengine` imports no provider, and liveness and the interrupt marker reach it only through `SessionProber` and `ActivityReader`.
  `cmd/lyx/seamsignature_test.go` passes.
  Not a defect.
- Wait loop cursor: `ParseSessionSignals` returns `consumed` stopping before the earliest line of the trailing run of stamps, and the shadow cursor starts at `PromptOffset`.
  `run.offset`, `ParseEvents`, `EventKind` and every wait-loop verdict are unchanged.
  Not a defect.
- Card 11 background shells: `payloadShell` gates the new behavior, so a shell without `SignalPayload` keeps the `background_shell_wait_min` expiry.
  A gated run with every output present and only unawaited payload shells returns `OutcomeDone` on the first tick.
  Not a defect.
- Test economy: every `test-budget.yaml` row rises by exactly the top-level functions its card adds (`shuttleengine` +5, `claudeengine` +5, `shuttlecli` +2, `orchengine` +1, `battenshed` +1).
  `TestBuildSettings_StopHook` became `TestBuildSettings_RecordingHooks` in place, count-neutral.
  Not a defect.
- Per-card mechanical checks: cards 1 and 2 add no rename or delete group, and their documents and notices are pinned byte for byte in `settings_test.go`.
  The python hook's allow and deny rows run under `sh` in `hookcmd_integration_test.go`, and `deniedRecoveryForms` gained `python` and `python3` with a self-test row.
  Not a defect.

### [MEDIUM] The wait loop's fact refresh cannot see a late interrupt or API-error marker

**Location:** `internal/shuttleengine/waitshadow.go:132`, with `internal/shuttleengine/sessionread.go` (`readSessionFacts`)
**Issue:** `logSessionState` calls `readSessionFacts(record, engine, signals, now)` with `signals` set to the bytes read since the shadow cursor.
`readSessionFacts` finds the newest turn start and turn end only among the signals it is handed, and reads `TurnStartInterrupt` and the `ActivityReader` API-error marker only for those.
At a liveness tick with no new bytes, `signals` is empty, so `facts.Interrupted` and `facts.APIError` come back false, and the refresh re-reads nothing but liveness.
The interrupt marker is written to the transcript after the `UserPromptSubmit` line, with no hook of its own.
The tick that folds the turn start therefore reads the transcript before the marker exists, and no later tick looks again.
The wait loop's fold stays `busy` `turn` for an interrupted turn while `lyx shuttle state`, which folds the whole file in one call, reports `idle-stalled` `interrupt`.
The loop classification `running` agrees with `busy`, so no disagreement is logged and the gap stays invisible.
The same holds for a transcript API-error marker that lands after its Stop line was folded.
`waitshadow_test.go` has no row for either marker, so nothing pins the refresh.
The plan (card 13) wants the shadow to fold "the same signals `lyx shuttle state` folds", and shadow mode exists to check the state against the loop before any consumer acts on it.
**Fix:** Keep the newest turn start and the newest turn end the shadow has folded on `sessionShadow`, and pass them to the fact read at each refresh (new signals first, else the remembered ones).
Add a `waitshadow_test.go` row where the interrupt marker appears after the turn start was folded and a later tick logs `idle-stalled` `interrupt`.

### [LOW] Comments cite plan artifacts instead of stable identifiers

**Location:** `internal/shuttleengine/claudeengine/signals.go:3`, `internal/shuttlecli/smoke_turns_test.go:378`
**Issue:** `TestSmokeTurnsSessionHooks` says it "pins card 3's parse", and the `signals.go` file comment says the payload-shape facts live there "per the provider-seam containment decision".
The plan and its card numbers go away with the task, so both sentences stop pointing at anything.
`scribe:code-quality` bars comments that reference the current task, and a name cited in a comment is a stable identifier in backticks.
**Fix:** Say `ParseSessionSignals` in the test comment, and name `PATTERN-shuttle-provider-seam` in the file comment.

### [LOW] TestRunner_SessionState is serial with no stated reason

**Location:** `internal/shuttleengine/session_test.go:292`
**Issue:** `PATTERN-test-speed` has every test run `t.Parallel` unless a comment names the global state that prevents it.
`TestRunner_SessionState` builds its own fixture and clock and touches no process-global state, and its neighbor `TestRunner_LoadSkills_GuardStrands` builds the same fixture under `t.Parallel`.
**Fix:** Call `t.Parallel()` at the top of the test.
