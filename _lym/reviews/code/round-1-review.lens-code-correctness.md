```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, which adds `readinessState` and routes `sessionReadiness` through it.

**List (function or branch, case run, verdict)**

- `readinessState`, first read returns a non-unknown state (a signal lies past the offset): returned as is. Correct.
- `readinessState`, first read returns unknown with a cause other than `no-signal`, for example `events-unreadable` or `no-state`: the guard at `readiness.go:131` returns it unchanged. Correct for `events-unreadable`, since a refold would read the same failing file. `no-state` is covered by the finding below.
- `readinessState`, `PromptOffset == 0` with `no-signal`: returns unchanged and does no second read. Correct.
- `readinessState`, the target case. The offset is N, the file holds `[session-start, turn-start, turn-end]` up to N, and nothing lies past N. The first fold sees an empty slice, and `settle` takes the `!f.hasNewest` branch (`sessionstate.go:292`), giving unknown/`no-signal`. The refold with the offset at 0 reads the load turn's end and returns idle. Correct.
- `readinessState`, offset past the file length: `readRunSessionState` clamps the slice (`sessionread.go:92`), so the first read is `no-signal` and the refold reads the whole file. Correct.
- `readinessState`, stale unmatched turn start before the offset: the refold yields busy/turn, so `judgeReadiness` holds with "a turn is running". Before the diff, `no-signal` gave the pane fallback. `unmatchedTurnStart` in `sessionReadiness` reads `sc.eventsPath`, and the manifest does not show whether it honours the offset. A start left unmatched before the offset can now hold the send until the hold releases (a pane idle for `turnStartIdleOverride`, or an interrupt). The hold exists for this case, so I rate it correct but unproven.
- `sessionReadiness`, edge of the refold: `rs` is passed by value, so setting `PromptOffset = 0` mutates only the local copy and the caller's `sc.state` is unchanged. Correct.
- `sessionReadiness`, Debug-log cause: after the refold, a still-unknown state reports the cause of the whole-file fold (`no-signal`, or `no-state`/`events-unreadable`). Not an error.
- `hookStateSkillEngine.IdleSession`, now `e.hooks.IdleSession(capture)` only: correct as a test fake. It has no effect on production code.

**Findings**

1. Severity LOW, class `design`.
   - Location: `internal/shuttleengine/readiness.go:131`.
   - Summary: the refold triggers only on cause `no-signal`. If lines past the offset parse to signals but derive no state, `settle` returns unknown/`no-state` (`sessionstate.go:294`) and the pane fallback that caused the original failure remains. An example is a late-appended `session-end` or other non-state signal after the load turn.
   - Evidence: `sessionstate.go:292-295`. I did not check whether the parser emits non-state signals.
   - Fix: also refold on `SessionCauseNoState`, or state in the docs that only `no-signal` is covered.

2. Severity LOW, class `consistency`.
   - Location: `internal/shuttleengine/session.go:156`, `conditionedsend.go:66`, `hold.go:174`.
   - Summary: other readers (`Runner.SessionState`, `conditionedReading`, `continueAPIErrorStall`) still read through `readRunSessionState` bounded at the offset. For the same run right after the load turns, `SessionState` reports unknown/`no-signal` while `SessionIdle` and the send path report idle. This is deliberate for `Wait`-side readers; `conditionedReading` is a send-path reader that stays offset-bounded.
   - Evidence: `conditionedsend.go:66`.
   - Fix: none required if the split is intended. Otherwise say so in the comment on `readinessState`.

Tool calls (2):
1. grep `SessionCauseNoSignal` in `internal/shuttleengine`: to find where `no-signal` is produced. My context lacked the fold's `settle`.
2. Read `sessionstate.go:225-345`: to see the `settle` branches (`no-signal`, `no-state`, `events-unreadable`).
