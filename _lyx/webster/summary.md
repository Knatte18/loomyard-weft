# Pause sends Esc only into a running turn

Both batches recorded `status: done`.

## Batch 01-running-turn-predicate (head `2e174f695`)

- Added `SessionState.RunningTurn`, shared by send readiness.
- Declared the `InterruptGuard` and `RewindPickerReader` capabilities and implemented them in Claude, with the captured picker fixture.
- Let shuttlefake script the running-turn and picker readings and the settle.
- Guarded every interrupt key by the pane and closed an open rewind picker once, in interrupt and send, with `lyx shuttle interrupt`'s help.
- Made `Runner.Inject` refuse the interrupt key and added `PATTERN-guarded-interrupt`.
- Deviations: `internal/shuttleengine/interruptguard.go`, `send.go`, `engine.go`, `fakes_test.go`, `run_inject_test.go` and `pattern/PATTERN-guarded-interrupt.md`, outside the declared file-ops.

## Batch 06-session-busy-reason (head `931c6ab6f`)

- `SessionBusyError` carries a held send's readiness reason.
- A CLI test pins the guard failures of `lyx shuttle interrupt` and `send`.
- The pause interrupts only a running turn, settles before a guarded retry, lists background work, and resume reports the guard's reason.
- The orch watcher waits the settle across ticks before a guarded second Esc and logs refusals.
- Deviation: `internal/pauseengine/pause.go`, outside the declared file-ops.
