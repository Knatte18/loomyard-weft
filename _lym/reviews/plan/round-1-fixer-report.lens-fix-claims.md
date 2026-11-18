```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked whether each sentence the fix added or rewrote holds against the code at HEAD. One finding.

**MEDIUM.** Card 14's new load-turn paragraph, and card 16's rewritten start-path case, both claim the usage-limit (or refused-resume) turn end is caught "before the load hold counts it" and "ends the load hold uncounted". The code does not support either claim.
- `readLoadTurn` reads parser signals first. `internal/shuttleengine/loadturn.go:212-221` handles a `SessionSignalAPIErrorTurnEnd` there by calling `watch.apiErrorRead`, which does `w.errors++` at `loadturn.go:96`.
- That happens before the event loop at `loadturn.go:228-240`, which is the only place the cards hook in through `TurnEndActivity`.
- Claude emits exactly that signal for a StopFailure hook, with no usage-limit reading: `internal/shuttleengine/claudeengine/signals.go:163-168`.
- So on claude the limited load turn end is counted on the signal path before any `TurnEndActivity` check runs. If no `EventStop` or `EventWaiting` line follows the StopFailure, the switch or refused-resume check is never reached at all.
- The cards need to also cover the `SessionSignalAPIErrorTurnEnd` branch, or make the signal path defer to the event read. Otherwise the stated ordering does not hold.

Claims that hold:
- **Card 3:** nine launching members are now listed under Edit.
- **Card 7:** `enterSession` already holds `location`, at `watchdog.go:333`, before its `SessionNamers()` call at line 359.
- **Card 12:** `strandRemovable` has exactly two callers besides the stop verb (`stop.go:148`): `attach.go:236` and `stepstop.go:99`.
- **Card 2:** `[]string{"account_pins", "accounts"}` matches `MachineKeys`' sorted order (`layers.go:190`).
- **Card 14:** `settleLoadTurn` passes `loadSkillTurn`'s error through unchanged (`run.go:1042-1044`), so it needs no Edit entry.

Tool calls:
1. Grepped reedcli for `SessionNamers` and `enterSession`, `settleLoadTurn`'s returns, claude's `SessionSignalAPIErrorTurnEnd` emitter, `strandRemovable` callers and `MachineKeys` sorting. My context lacked these call sites.
2. Read `signals.go:150-170` and `loadTurnWatch.apiErrorRead`. My context lacked whether the signal path counts errors and when claude emits that signal.
