```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the `session_test.go` change (the `seeded` field, `PromptOffset` wiring and three new rows) through the fix-code-correctness lens.

no findings

Traces, each run by hand against `readinessState` (`readiness.go:129`) with the fallback removed:
- **Row "reads the load turn's end":** the offset equals the file length, so the fold past it gives unknown with cause no-signal and a non-zero offset. With the fallback in place, the whole-file fold gives idle at `endedAt`, so the probe is `idleAtEnd` (`TurnEndedAt` set).
  - With the fallback removed, the state stays unknown and the pane fallback returns `IdleProbe{Idle: true, NoTurn: true}`, which has no `TurnEndedAt`. The row fails, so it pins the defect.
- **Row "keeps a stale unmatched turn start held":** with the fallback, the whole-file fold reads busy with a running turn, and the hold holds on the first poll, giving "a turn is running".
  - With the fallback removed, the unknown state takes the pane reading, which is idle. The row fails, so it pins the defect.
- **Row "zero offset":** `PromptOffset` is 0 and the events file is empty. The guard returns the unknown state and the pane reading gives `Idle` and `NoTurn`.
  - This row passes with or without the fallback. It pins the zero-offset guard's visible result, and the review asked for it.
- **Wiring:** `PromptOffset: int64(len(tt.seeded))` is 0 for every pre-existing row, so their behaviour is unchanged. `events := tt.seeded` is rewritten to the file on the first probe, so the offset matches the file length.
- **Gate:** the tier0 gate for `./internal/shuttleengine` passed (1280 passed, 0 failed), so the new rows ran and pass.
- **Callers:** no signature changed. Only test code changed.

Further tool calls: none (0 of 6).
