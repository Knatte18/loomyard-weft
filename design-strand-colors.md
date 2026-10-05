# Each strand role gets its own color

`strand-colors` · Note · enhancement, reed, shuttle

## Problem

With a run live, the driver pane and the agents it spawns (Webster, Burler, Bouncer judge, Describe and the rest) look alike in tmux and in Claude Code, so it is often hard to see which session is which.

## Idea (operator, 2026-10-05)

Give each spawned session a color with Claude Code's `/color` command, so the driver stands apart from the agents it spawns.
Open question: one color per loom segment (Discussion, Plan, Webster, review, Describe, landing) rather than only driver against the rest; the operator would like that.

## Open

- Where the color is set: by the spawning code through reed when it starts the strand, or by the agent's own launch prompt.
- Whether the tmux pane border or the strand tabs of `reed-status-tabs` should carry the same color, so the reed layout matches the sessions.
