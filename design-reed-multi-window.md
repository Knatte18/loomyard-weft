# reed: a strand in its own tmux window

`reed-multi-window` · Task · feature, reed, shuttle · running · Loom-Preflight

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

## Was `extra-agent-strand`

The operator wants extra Claude strands beside the orch in the prime, such as one for exploring the repo, added by the orch on request (operator, 2026-10-06).

Tried 2026-10-06: `lyx reed add --name explore --cmd claude` runs a bare `claude`, so the pane title is `ly:explore` but the Claude session gets no `--name`, and reed records no session id, so the watchdog's session-name repair (`internal/reedengine/namerepair.go`) skips it.

Every agent in lyx runs through shuttle (operator), so this needs a shuttle engine extension: open a named LLM session in a strand with no turn and no prompt, started with the strand name and its session id recorded, so the watchdog keeps the name across a compact.
The verb is not in reed, which shuttle depends on.
Shuttle now has a CLI of its own (`internal/shuttlecli`: `run`, `send`, `await`, `interrupt`, `state`), so the verb could live there.

Open: where such a strand goes in the reed layout relative to the orch, and how several share the space.

With strands in their own windows, an extra agent strand gets its own window; this answers its layout question.
