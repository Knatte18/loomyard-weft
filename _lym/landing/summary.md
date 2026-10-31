# shuttleengine: no hold notice for a turn the operator typed in the pane

A held turn end whose turn began with an instruction the operator typed into the agent's pane now sends the parent no hold notice.
Before, an operator talking to a held agent made lym tell the orch the agent "ended a turn without its output files and is held", and the notice did not say who the agent had talked to.
A held end of a turn lym started, by the run's prompt, a re-prompt, a continue line or a creel message, is notified as before.

## Design

- A turn start past the run's prompt offset is the operator's when it carries a typed instruction, is not the run's prompt turn start, and its text is not in the run's sent-texts file.
  This is the reading the carry-over of operator instructions already made, so both consumers now share it: `runTurnStarts` in `carryover.go` produces it, and `operatorInstructions` is built on it with the same result as before.
- The turn a held end closes began at the newest turn start whose line lies before the turn end's offset.
  The events file holds the lines in order, so no in-memory tracking is needed and the reading survives an attach.
  With no such turn start, the turn is not the operator's and the notice is sent.
- `Wait`'s held branch gains one case beside the usage-limit and background-wake skips.
  It logs one Info line, sends nothing, defers nothing and leaves `RunState.NotifiedOffset` alone.
  The hold, the wait mark and the deadline are unchanged.
- `positionedTurnStarts` in `wait.go` now returns each turn start's signal with its end offset; `turnStartOffsets` is a thin wrapper over it.
- The reading stays provider-neutral through the optional `SessionSignalParser`.
  An engine without it, or an unreadable events file, reads no operator turn and notifies as before.
- The package doc's hold-notice paragraph, which said shuttle cannot tell an operator's typed text from a parent's answer, is rewritten.

## Bounds

- A send whose sent-texts write failed reads as the operator's, so that turn's held end sends no notice.
  The failed write is already logged at Warn, and the next held end of a turn lym starts is notified.
- An operator who types the exact text of an earlier lym send reads as lym's send, and that turn's held end is notified.
- A turn start whose Raw line the events file does not hold is not read.
  The test engine in `carryover_test.go` now sets `Raw` on its signals for this reason.

## Tests

Rows in the tier0 `TestWait_HeldTurnEndNotice` table, on the fake engine and an injected clock, pin:

- an operator-typed turn sends no notice and leaves the offset;
- a deferred run after an operator turn neither queues nor sends;
- the prompt turn, a turn start with no instruction and a turn a recorded send started are notified around an operator turn.

I did not run the tests here; run them with `lym gate test ./internal/shuttleengine`.
