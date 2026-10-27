# An instruction typed into a pane is lost when the tackler re-spawns the row

`pane-input-lost-on-respawn` · Note · bug, shed, shuttle, reed

## Seen

An instruction the operator typed into a worker's tmux pane was lost when the tackler re-spawned that row's session: the new session started from the row's prompt alone, and nothing carried the typed line over.

## Idea

When a row is re-spawned, carry over what the operator said to the old session, for example by reading the operator's turns from the old transcript, or by routing operator input through creel so it is recorded and re-delivered to the new session.
At the least, warn the operator in the pane or by a notice that the session was replaced and an unanswered instruction may be lost.
