# Decision record — hold-notice-operator-turn

## Goal

A held turn end of an autonomous shuttle run sends its parent no hold notice when the turn began with an instruction the operator typed into the agent's pane.
A held turn end of a turn that lym started, by the run's prompt, a re-prompt, a continue line or a creel message, is notified as today.

## Scope

- `internal/shuttleengine`: Wait's held-turn-end branch skips the notice for an operator-started turn, and the reading that tells an operator's typed instruction from lym's own sends is shared with the carry-over of operator instructions.
- The package doc's hold-notice paragraph, which today says shuttle cannot tell an operator's typed text from a parent's answer.
- Out of scope: the notice's text, creel's delivery path and the engines' turn-start parsing, which already yield a turn start's typed instruction.

## Decisions

### D1 — The sent-texts record decides who started a turn

A turn start past the run's prompt offset is the operator's when it carries a typed instruction (`SessionSignal.Instruction` non-empty), is not the run's prompt turn start, and its text is not in the run's sent-texts file.
This is the reading the carry-over of operator instructions already makes: every verified send into a run with a record, creel mail and the run's own re-prompts and continue lines included, appends its text to that file before typing.
A turn start with no instruction, a task notification or a slash command, is not the operator's.
Rationale: one reading for both consumers, and no new record.
Bound: a send whose sent-texts write failed reads as the operator's, so that turn's held end sends no notice; the failed write is already logged at Warn, and the next held end of a turn lym starts is notified.

### D2 — The held turn's start is the newest turn start before the turn end

The turn a held turn end closes began at the newest turn start whose line lies before the turn end's offset.
With no turn start read past the prompt offset before it, the turn is not the operator's and is notified as today.
Rationale: the events file holds the turn start lines in order, so the reading needs no in-memory tracking and survives an attach.

### D3 — An operator-started held turn end sends no notice and leaves the notified offset

Wait's held branch gains one case beside the usage-limit and background-wake skips: when a notice would be pending and the turn is the operator's, it logs one Info line, sends nothing, defers nothing and leaves `RunState.NotifiedOffset` alone.
The hold, the wait mark and the deadline are unchanged.
A later held turn end of a turn lym or mail started is notified as any other.
Bound: the skip lets through only held ends of operator-typed turns; it never skips an interactive, quiet-hold or no-notifier decision, which already send nothing.

## Constraints

- `PATTERN-shuttle-provider-seam`: the reading stays provider-neutral in `internal/shuttleengine`, through the optional `SessionSignalParser` capability; an engine without it reads no operator turn, so its held ends are notified as today.
- `PATTERN-test-speed`: the behavior is pinned as rows of the existing tier0 hold-notice table test, on the fake engine and an injected clock.
- `PATTERN-tolerant-records`: no record format changes.

## Auto-mode assumptions

- The operator's conversation with a held agent is not a stall the parent must act on, so suppressing the notice for each such turn is wanted even across several operator turns in a row.
- Creel mail reaches an agent through shuttle's verified send into its run, so it lands in the sent-texts file; mail a hook adds to a running turn starts no turn.

## Open risks

- A run whose events file is unreadable at the held turn end reads no operator turn and notifies as today.
- An operator who types the exact text of an earlier lym send into the pane reads as lym's send, and that turn's held end is notified.

## Acceptance criteria

- D1, D2, D3: a held turn end whose turn began with an instruction typed into the pane and absent from the sent-texts file sends no notice and leaves `RunState.NotifiedOffset` alone.
- D1, D3: a held turn end whose turn began with a text the sent-texts file holds is notified.
- D1, D2: the held turn end of the run's prompt turn is notified, and so is one whose turn start carries no instruction.
- D3: after an operator-started held turn end, the held end of a later turn lym started is notified.
- D1: the carry-over of operator instructions reads the same instructions as before.
