# An idle interactive run saves its work and pauses, to go on another time

`interactive-idle-park` · Note · enhancement, shed, shuttle, batten

## Seen (operator, 2026-10-31)

An interactive run (task `interactive-recipe`) waits for the operator in its pane, and has dobby's quiet notice and shuttle's idle timeout off while it does.
Since lym runs the agent, it can watch how long the agent has waited.

## Idea

When the agent has been idle for a set time, lym asks it to write down what it has, commit it, and end its session; the run pauses, and the operator picks it up another time with the agent resumed from that state.
