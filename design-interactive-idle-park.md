# An idle interactive run saves its work and pauses, to go on another time

`interactive-idle-park` · Note · enhancement, shed, shuttle, batten

## Seen (operator, 2026-10-31)

An interactive run (the `sampler` recipe, landed by task `interactive-recipe`) waits for the operator in its pane at the `Sampler-Session` row, and has dobby's quiet notice and the step watchdog off while it does.
The row waits with no timeout until the agent runs `lym loom sampler-done` at the operator's word.
Since lym runs the agent, it can watch how long the agent has waited.

## Idea

When the agent has been idle for a set time, lym asks it to write down what it has, commit it, and end its session; the run pauses, and the operator picks it up another time with the agent resumed from that state.
