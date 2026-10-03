# Merriam and the driver reflect on their own session and file issues at the end

`agent-self-reflect` · Notes · design

Seen 2026-10-03: no friction from today's driven runs reached GitHub.
Friction-Reflect spawns its reflection agent only under `lyx loom run` (`internal/loomcli/run.go`, `reflectFrictionRow`).
Under `lyx shed step`, which every ly-drive driver uses, it skips, and leaves the notes for an operator-gated filing that autonomous drivers never do ("Self-report draft: none filed").
Batten's teardown then deletes `_lyx/loom/friction/` with the pair.
The friction worth filing today (Webster's Delete ordering, a failed batch that cannot be redone, recoveries that need denied commands) was found and filed by the hub by hand.

Idea (operator): the agents that matter, at least Merriam (webster's Master) and the driver, reflect on their own session at the very end and file what they hit as GitHub issues themselves.
This is the model millhouse uses: `mill-self-report` runs at end-of-work in its orchestrating agents, scans its own session context for tool failures, refusals, prompt mismatches and surprises, and files the candidates with `gh` in auto mode.
An agent reflecting on its own context sees what a cold aggregator over notes never sees.
Open: dedupe against open issues before filing; the reflection must run before batten tears the pair down; how this relates to the existing note-writing and Friction-Reflect (replace, or keep the aggregator for agents that leave notes but do not self-report).
Related: `batten-llm-friction` (#274, teardown deletes the notes).
