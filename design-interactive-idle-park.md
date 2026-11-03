# An interactive agent idle for a set time gets its session compacted

`interactive-idle-park` · Note · enhancement, shed, shuttle, batten

## Seen (operator, 2026-10-31)

An interactive run (the `sampler` recipe, landed by task `interactive-recipe`) waits for the operator in its pane at the `Sampler-Session` row, and has dobby's quiet notice and the step watchdog off while it does.
The row waits with no timeout until the agent runs `lym loom sampler-done` at the operator's word.
Since lym runs the agent, it can watch how long the agent has waited.

## Idea (operator, 2026-11-03)

An auto-compact mode, in place of the earlier idea of parking the run:

- When an interactive agent has been idle for a set time, lym compacts its session, the way `lym orch distill` compacts the orch, and the agent stays in its pane with the run still at its row.
- The operator comes back to the same session with a small context, so nothing is ended, committed or resumed by hand.
- The idle time is a config key; off means never.

The earlier idea, writing down the state, committing it, ending the session and pausing the run, is dropped: it is too specific to be worth its cost.
