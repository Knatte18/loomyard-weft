# Merriam and the driver reflect on their own session and file issues at the end

`agent-self-reflect` · Notes · design

Seen 2026-10-03: no friction from today's driven runs reached GitHub.
Friction-Reflect spawns its reflection agent only under `lyx loom run` (`internal/loomcli/run.go`, `reflectFrictionRow`).
Under `lyx shed step`, which every ly-drive driver uses, it skips, and leaves the notes for an operator-gated filing that autonomous drivers never do ("Self-report draft: none filed").
Batten's teardown then deletes `_lyx/loom/friction/` with the pair.
The friction worth filing today (Webster's Delete ordering, a failed batch that cannot be redone, recoveries that need denied commands) was found and filed by the hub by hand.

Idea (operator): the agents that matter, at least Merriam (webster's Master) and the driver, reflect on their own session at the very end and file what they hit through `lyx selfreport create`, lyx's own verb for exactly this, never through `gh`.
This is the model millhouse uses: `mill-self-report` runs at end-of-work in its orchestrating agents, scans its own session context for tool failures, refusals, prompt mismatches and surprises, and files the candidates in auto mode.
An agent reflecting on its own context sees what a cold aggregator over notes never sees.
Decided (operator, 2026-10-03): no approval gate on filing, as in millhouse: everything is filed, and filtering happens when issues are pulled back into the board.
That removes the reason for both skips: `reflectFrictionRow` reflecting only under `run` ("the reflection agent files public issues itself"), and ly-drive's `## Self-report` filing only on operator approval and nothing in autonomous mode.
Open: dedupe against open issues before filing, or leave it to the intake filter; the reflection must run before batten tears the pair down; how this relates to the existing note-writing and Friction-Reflect (replace, or keep the aggregator for agents that leave notes but do not self-report).
Related: `batten-llm-friction` (#274, teardown deletes the notes).
