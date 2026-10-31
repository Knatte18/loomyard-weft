# No shuttle hold notice for a turn the operator started in the pane

When an autonomous run's agent ends a turn without its output files, shuttle sends the parent a hold notice.
That notice also went out when the turn was the operator's own conversation with the agent in its pane, so the orch received a stall report it could not act on and had to ask the operator what happened.

A held turn end now sends no notice when its turn began with an instruction the operator typed.
The check reuses the reading the carry-over of operator instructions already makes.
A turn start past the run's prompt offset counts as the operator's when it carries a typed instruction, is not the run's prompt, and its text is not in the run's sent-texts file.
Every verified send into a run writes its text to that file before typing, including creel mail, gate re-prompts and the API-error continue line.
The turn a held end closes began at the newest turn start before the turn end's line in the events file.

## Changes

- `internal/shuttleengine/carryover.go`: `runTurnStarts` returns each turn start past the prompt offset with its offset, and carries its instruction only when the operator typed it.
  `operatorInstructions` now filters that list, so the carry-over reads the same instructions as before.
- `internal/shuttleengine/wait.go`: the turn-start offset reading becomes `positionedTurnStarts`, which keeps each signal beside its offset, and `turnStartOffsets` maps over it.
  Wait's held-turn-end branch adds a case beside the usage-limit and background-wake skips: an operator-started turn logs one Info line, sends nothing, defers nothing and leaves `RunState.NotifiedOffset` alone.
- `internal/shuttleengine/hold.go`: `operatorStartedTurn` decides whether a held turn end's turn was the operator's.
- `internal/shuttleengine/doc.go`: the hold-notice paragraph used to say shuttle cannot tell an operator's typed text from a parent's answer. It now states the new rule.

## Design decisions

- The held end of a later turn started by lym or by mail is notified as before, so a real stall after an operator conversation still reaches the parent.
- A turn start with no typed instruction, such as a task notification or a slash command, does not count as the operator's, and its held end is notified.
- If writing the sent-texts file fails, that send reads as the operator's and its turn's held end sends no notice. The failed write is already logged at Warn, and the next held end of a turn lym starts is notified.
- An engine without the session-signal capability, and an unreadable events file, read no operator turn, so their held ends are notified as before.

## Tests

- New rows in `TestWait_HeldTurnEndNotice`:
  - An operator-typed turn sends no notice and leaves the notified offset.
  - Around an operator turn, three held ends are each notified: the prompt turn's, one from a turn with no instruction, and one from a turn a recorded send started.
- The fake session engine reads `START:<text>` as a turn start carrying a typed instruction.
- The carry-over test's fake engine now sets `Raw` on its turn starts, as production engines do, because the positioned reading matches signals to lines by it.
