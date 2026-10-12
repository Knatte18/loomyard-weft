# Card 3 — shuttlefake implements the pane-reading capabilities

**Create:**
- `plan:internal/testkit/shuttlefake#Engine.RunningTurn` -> `func (e *Engine) RunningTurn(capture string) bool`
- `plan:internal/testkit/shuttlefake#Engine.RewindPicker` -> `func (e *Engine) RewindPicker(capture string) bool`
- `plan:internal/testkit/shuttlefake#Engine.InterruptSettle` -> `func (e *Engine) InterruptSettle() time.Duration`

**Edit:**
- `internal/testkit/shuttlefake#Engine`
- `internal/testkit/shuttlefake#TestEngine_InertDefaults`
- `internal/testkit/shuttlefake#TestEngine_Overrides`

**Intent:**
D3 and D8 name the fake engine in `internal/testkit/shuttlefake` as an implementer of the running-turn and rewind-picker readings, so a consumer test can drive the guards card 4 adds.

- `Engine` gains three fields beside the other `...Fn` overrides: `RunningTurnFn func(capture string) bool`, `RewindPickerFn func(capture string) bool` and `InterruptSettleFn func() time.Duration`.
- The three methods answer their func when set; otherwise `RunningTurn` and `RewindPicker` answer false and `InterruptSettle` answers 0, matching the type's "inert unless scripted" contract.
  A running turn that defaults to false means a consumer interrupting through this fake gets card 4's `ErrNoRunningTurn` unless it scripts `RunningTurnFn`; no consumer interrupts through it today.
- The `var` block of compile-time assertions gains `shuttleengine.InterruptGuard` and `shuttleengine.RewindPickerReader`.
- `TestEngine_InertDefaults` asserts the three defaults: `RunningTurn("x")` false, `RewindPicker("x")` false and `InterruptSettle()` 0.
- `TestEngine_Overrides` gains a subtest "Guard readings" setting all three funcs and asserting each answer comes from its func: `RunningTurnFn` and `RewindPickerFn` echo whether the capture equals a scripted string, and `InterruptSettleFn` returns `2 * time.Second`.

**ImpactSummary:** Additive fields and methods on a test-only kit. The extra picker capture that card 4 adds to a send through this fake reads `CapturePaneFn`, and every consumer's `CapturePaneFn` is stateless (`claudeengine` start and corpus tests, `shuttlecli` send test), so none shifts.

**Commit:** 3: Let shuttlefake script the running-turn and picker readings
