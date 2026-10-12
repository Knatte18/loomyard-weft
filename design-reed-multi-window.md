# reed: a strand in its own tmux window

`reed-multi-window` · Note · feature, reed

Own-window strand anchoring: a `display` anchor that spawns a strand into its own switchable tmux window instead of a pane.
Operator 2026-10-12: wanted at some point, not now.
Later the same day the operator reconsidered, since there is more to show than the panes leave room for: an important strand always gets its own full-size window, which replaces zooming a pane in reed.
The unit is the producer, not the strand: a multi-LLM producer with two agents shows both together in one window (operator, 2026-10-12).
A producer with many agents (5–6) does not fit in one window, so each of those gets its own.
The strand's `display` anchoring setting carries all of this: a window shared by the producer's strands, or a window per strand.
In a shared window the strands split either by height or by width (side by side), set the same way.

## Operator, 2026-10-05: strands in depth, not stacked in height

Two strands stacked as panes in one window are unreadable on a small laptop screen.
Stack them in depth instead: one tmux window per strand, behaving like tabs within the run's terminal window.

- The driver always lives in window 0, with its `loom-status` strand beside it.
- Each agent the driver spawns (Discussion-Write, a Burler round, Webster's Master) gets its own window, named by its role, so the tmux status line lists what runs.
- The window closes when the agent ends, and the view falls back to the driver.
- **No focus stealing.** A new window must never pull focus away from a window the operator is typing in; an automatic switch on spawn is unwanted for that reason.
  Open: how a new window announces itself instead (status-line highlight, tmux activity flag, a switch only when the current window is idle).

## Dropped (operator, 2026-10-12)

- Cross-worktree columns (all worktrees in one tmux window, a column per worktree): today's one window per run is better.
- Independent per-window attach through tmux session groups: outdated.

## Related

- `internal/reedengine/window.go` (since a35c8c2b1): reed already opens detached, named, hidden tmux windows in a worktree's session running a `lyx` command; such a window is not a strand, so strand-per-window anchoring is still open.
