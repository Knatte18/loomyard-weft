# reed: a viewer on the newest agent or Driver window follows a new agent

An attached run window stuck on the Driver window while later agents spawned in windows of their own.
Reed picked the window only at attach, and a new own-window strand took the viewer only from the overview or from an all-dead agent window.

## What changes

`newAgentWindowSelected` in `internal/reedengine/ownwindow.go` now also selects a new agent window when the session's current window is either of these:

- the relevant window as listed before the new one opens (`relevantWindow`): the highest-index agent window holding a live pane, where attach and every automatic switch put the viewer;
- a window holding a live pane of an own-window strand whose role matches `agentname.RoleDriver`, so a viewer still parked on the Driver follows the run even once a newer agent is live.

A viewer on an older live agent window or on the shell is still never moved, so a window the operator picked by hand keeps them.
Focus strands, the overview and all-dead agent windows behave as before, and a selected window still carries no `@lyx_unseen`.
`newAgentWindowTakesViewerLocked` now lists the session's windows as well as its panes; a failed listing keeps the viewer, logged at Warn.

## Design decisions

- The switch stays one session-wide `select-window`: every client attached to a reed session shares its current window, so it moves each attached client.
- The Driver is found by the bound strand's role through `internal/agentname`, never by window name, and only an own-window driver counts, so a driver placed in the strands' window never pulls a viewer off the shell.
- A strand joining an existing shared group window opens no window and switches nothing, as before.
- Replacing the viewed agent now lands the viewer on the replacement: the removal falls back to the newest live agent, and the replacement then takes the viewer from there.

## Tests and docs

- `TestNewAgentWindowSelected` pins the rule over a table of current windows; the `internal/reedengine` row in `cmd/lyx/testdata/test-budget.yaml` rises by one for it.
- `TestWindowOrder_RealTmux` gains a subtest for following from the newest agent and from the Driver, keeps the viewer on an older agent instead of the newest one, and its replace subtest now expects the replacement's window.
- The focus-rule paragraph in `internal/reedengine/doc.go` states the widened rule.
