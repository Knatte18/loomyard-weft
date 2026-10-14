# reed: an attached run window follows the newest agent window

An attached run window used to stay on the Driver window while later agents spawned in windows of their own, because reed chose a window only at attach.
When a new own-window strand spawns, reed now moves the viewer to the new window if the viewer is on the previous newest live agent window or on a Driver window.
A viewer the operator moved to any other window by hand stays where it is.

## What changes

`newAgentWindowSelected` in `internal/reedengine/ownwindow.go` takes the session's windows as well as its panes.
Besides a Focus strand, the overview and an all-dead agent window, which already passed the viewer on, it now also takes the viewer from:

- the relevant window as listed before the new window opens, the highest-index agent window holding a live pane;
- a window holding a live own-window pane of a strand whose role matches `agentname.RoleDriver`, found by the new `driverWindows`.

An older live agent window and the shell keep the viewer, as before.
`newAgentWindowTakesViewerLocked` lists the session's windows for this, and a failed listing keeps the viewer where it is, as the existing reads do.

The focus-rule paragraph of `internal/reedengine/doc.go` is updated to match.

## Design decisions

- The switch stays one session-wide `select-window` decided from the session's current window. Every client of a reed session shares its current window, so it moves each attached client.
- The driver role is matched only through `internal/agentname`, and only that role is singled out, so no other role widens the rule.
- The rule reads reed state, window ids, strand roles and pane liveness only, never `@lyx_unseen`, `@lyx_hidden` or a wait mark.
- An operator who picks the newest agent window by hand cannot be told apart from one who stayed there, and is moved.
- Replacing the viewed agent now lands the viewer on the replacement's window when the removal falls back to the newest live agent. The tmux-tier replace subtest's expectation changes with it.

Out of scope: attach's own window selection, the removal and reconcile switches, a strand joining an existing shared window by a split, and dropping the LLM Driver.

## Tests

- `TestNewAgentWindowSelected` is a unit table over the viewer's window, including the driver window, an older live agent window and the shell.
- The tmux-tier focus subtests in `windoworder_integration_test.go` are extended for the newest-agent, driver, older-agent and replace cases.
- `internal/reedengine` is raised by one in `cmd/lyx/testdata/test-budget.yaml`.

Not run by hand: `lyx gate test --tags tmux ./internal/reedengine` against a live run window, watching it follow a spawn.
