# Card 2 — Provider-neutral session-start sources and the idle-fresh fold state

**Create:**
- `plan:internal/shuttleengine#SessionStartSource` -> `type SessionStartSource string`
- `plan:internal/shuttleengine#SessionStartStartup` -> `const SessionStartStartup SessionStartSource = "startup"`
- `plan:internal/shuttleengine#SessionStartResume` -> `const SessionStartResume SessionStartSource = "resume"`
- `plan:internal/shuttleengine#SessionStartClear` -> `const SessionStartClear SessionStartSource = "clear"`
- `plan:internal/shuttleengine#SessionStartCompact` -> `const SessionStartCompact SessionStartSource = "compact"`
- `plan:internal/shuttleengine#SessionIdleFresh` -> `const SessionIdleFresh SessionStateName = "idle-fresh"`
- `plan:internal/shuttleengine#SessionCauseSessionStart` -> `const SessionCauseSessionStart = "session-start"`
- `plan:internal/shuttleengine/claudeengine#sessionStartSource` -> `func sessionStartSource(source string) shuttleengine.SessionStartSource`

**Edit:**
- `internal/shuttleengine#SessionSignal`
- `internal/shuttleengine#SessionSignalSessionStart`
- `internal/shuttleengine#SessionFold.applySignal`
- `internal/shuttleengine#judgeReadiness`
- `internal/shuttleengine#loopAgreesWithState`
- `internal/shuttleengine/claudeengine#signalFromPayload`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/shuttlecli#shuttleCLI.stateCmd`
- `internal/shuttleengine/sessionstate_test.go`
- `internal/shuttleengine/waitshadow_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/claudeengine/signals_test.go`
- `internal/orchengine/watcher_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/orchengine/doc.go`
- `docs/overview.md`

**Intent:**
D3.
`SessionSignal.Source` changes type from `string` to `SessionStartSource`, the provider-neutral source.
`claudeengine`'s new `sessionStartSource` maps Claude's `startup`, `resume`, `clear` and `compact` to the four constants and any other or missing string to the empty source;
`signalFromPayload` sets `Source` through it, so `shuttleengine` keys on no Claude string (`PATTERN-shuttle-provider-seam`).

The fold gains `SessionIdleFresh` with cause `SessionCauseSessionStart`.
In `SessionFold.applySignal` a session-start signal sets the derived state to idle-fresh when its source is `SessionStartResume` or `SessionStartClear`, or when it is `SessionStartStartup` and the derived state is not a running turn (busy with cause turn).
It still records `sessionStartAt` for every source.
A session start that sets idle-fresh becomes the newest signal; one with source `SessionStartCompact`, the empty source, or a `startup` start beside a running turn leaves the derived state and the newest signal unchanged, so an automatic compaction inside a turn keeps it busy and an idle-notice contradiction check is not masked.
`settle`'s precedence is unchanged: a dead or unproven liveness and an unreadable events file still outrank idle-fresh.
Update `SessionSignalSessionStart`'s doc, which today says a fold does not move on it.

`judgeReadiness` reads idle-fresh like idle-done: ready, subject to the same too-short, no-box, unmatched-turn-start and draft checks.
`loopAgreesWithState` and `Watcher.logIdleStateDisagreement` treat idle-fresh as an idle state: the held loop agrees with it as it does with idle-stalled and asking, the running and waiting loops do not, and the watcher's not-idle probe beside idle-fresh warns.
`lyx shuttle state`'s help lists idle-fresh among the states, as does `docs/overview.md`'s shuttle row; `shuttleengine/doc.go`'s fold and readiness sections, `claudeengine/doc.go` and `orchengine/doc.go`'s disagreement paragraph name it.

Tests, extending existing tables:
`TestSessionFold` gains the Fold criterion rows (`startup` alone reads idle-fresh; `SessionEnd(clear)` then `SessionStart(clear)` reads idle-fresh; `resume` after an unmatched turn start reads idle-fresh; `startup` after an unmatched turn start stays busy; `compact` inside a running turn stays busy; an empty source changes nothing; a dead or unproven liveness outranks idle-fresh);
`TestParseSessionSignals` gains rows over `testdata/hooks/events.jsonl` expecting each mapped source, and an unknown source string reading the empty source;
`TestRunner_SessionIdle_ReturnsScriptedClassification` gains the Readiness rows (idle-fresh with an empty box is idle; with a draft, a too-short pane or no box it is held with that reason);
`TestWait_LogsSessionStateBesideItsClassification` and the watcher's disagreement table gain an idle-fresh row.
The #549 case: `TestRunner_Start_TypesTheSegmentColorAfterStartup` and `TestStartup_SkillLoadTurns` each gain a row whose events file holds only a `SessionStart(startup)` line, with a live fake prober and a pane the pane fallback would refuse, and whose color and skill-load sends still go out from the hook state.
The `sessionFakeEngine` in `fakes_test.go` parses a source after `SESSIONSTART:`.

**ImpactSummary:** a started session with no turn yet now reads idle-fresh instead of unknown, so the color send, the skill-load send and the orch idle probe read the hook state where they fell back to the pane; every `Source` reader and the two idle-state enumerations move with it.
