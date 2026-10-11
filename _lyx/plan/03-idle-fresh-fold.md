# Card 3 — The idle-fresh fold state a session start sets

**Create:**
- `internal/shuttleengine#SessionIdleFresh`
- `internal/shuttleengine#SessionCauseSessionStart`

**Edit:**
- `internal/shuttleengine#SessionSignalSessionStart`
- `internal/shuttleengine#SessionFold.applySignal`
- `internal/shuttleengine#judgeReadiness`
- `internal/shuttleengine#loopAgreesWithState`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/shuttlecli#shuttleCLI.stateCmd`
- `internal/shuttleengine/sessionstate_test.go`
- `internal/shuttleengine/waitshadow_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/claudeengine/testdata/corpus/hook-capture/case.yaml`
- `internal/orchengine/watcher_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/orchengine/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/shuttleengine#SessionStartStartup`
- `internal/shuttleengine#SessionStartResume`
- `internal/shuttleengine#SessionStartClear`
- `internal/shuttleengine#SessionStartCompact`

**Intent:**
D3's fold and readiness half, keyed on the provider-neutral sources card 2 adds.

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
the `hook-capture` corpus case's `states` entries after its `startup`, `clear` and `resume` `SessionStart` lines change to idle-fresh with cause session-start, and its `compact` entry stays as card 1 recorded it, so `TestCorpus_ReplaysEachTurnEnd` replays the real lines through the new rule;
`TestRunner_SessionIdle_ReturnsScriptedClassification` gains the Readiness rows (idle-fresh with an empty box is idle; with a draft, a too-short pane or no box it is held with that reason);
`TestWait_LogsSessionStateBesideItsClassification` and the watcher's disagreement table gain an idle-fresh row.
The #549 case: `TestRunner_Start_TypesTheSegmentColorAfterStartup` and `TestStartup_SkillLoadTurns` each gain a row whose events file holds only a `SessionStart(startup)` line, with a live fake prober and a pane the pane fallback would refuse, and whose color and skill-load sends still go out from the hook state.
The `sessionFakeEngine` in `fakes_test.go` parses a source after `SESSIONSTART:`.

**ImpactSummary:** a started session with no turn yet now reads idle-fresh instead of unknown, so the color send, the skill-load send and the orch idle probe read the hook state where they fell back to the pane; the two idle-state enumerations move with it.
