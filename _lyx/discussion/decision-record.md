# Decision record — reed-window-follow

## Goal

An attached run window follows the newest agent window: when a new own-window strand spawns, reed moves the viewer to its window if the viewer is on the previous newest live agent window or on the Driver window.
A viewer the operator has moved to any other window by hand stays where it is.

## Scope

- In: the spawn-time focus rule in `internal/reedengine` (`newAgentWindowSelected` and its caller in `openOwnWindowLocked`), its unit and tmux-tier tests, and the focus-rule paragraph of `internal/reedengine/doc.go`.
- Out: attach's own window selection, which already picks the newest live agent window, and the removal and reconcile switches, which already move only a viewer on an ending window.
- Out: a strand joining an existing shared group window by a split; it opens no window, so no switch happens, as today.
- Out: dropping the LLM Driver (`orch-split`).

## Decisions

### Follow the newest agent window at spawn

Decision: `newAgentWindowSelected` also takes the viewer when the session's current window is the relevant window before the new window opens, that is the highest-index agent window holding a live pane (`relevantWindow` with nothing ending).
Rationale: that window is where attach and every automatic switch put the viewer, so a viewer still there has not moved by hand and is following the run.
Bound: a viewer on an older live agent window or on the shell is still never moved, so a hand-picked window keeps the viewer; an operator who picks the newest agent window by hand is indistinguishable from one who stayed and is moved.

### The Driver window always passes the viewer on

Decision: a viewer on a window holding a live pane bound to a strand whose role matches `agentname.RoleDriver` (`agentname.MatchesRole`) is also moved to the new agent window.
Rationale: the operator named the Driver window as the one runs stick on; the Driver spawns first, so it stops being the newest agent window as soon as a second live agent exists, yet a viewer still parked there is following the run.
Bound: only the driver role is singled out, by its name; no other role, and no window option, widens the rule.

### Selection stays session-wide

Decision: the switch stays one `select-window` on the session, as today, deciding from the session's current window.
Rationale: every client attached to a reed session shares its current window, so moving the session's current window moves each attached client, and a client on another window cannot exist in one session.

### Existing guarantees kept

Decision: a Focus strand still always takes the viewer, the overview and all-dead agent windows still hand it on, the overview window is never selected at spawn, and a selected window still carries no `@lyx_unseen`.
Rationale: the change only widens which windows pass the viewer on.

## Constraints

- `PATTERN-wait-mark-display-only`: the rule decides from reed state, window ids and pane liveness only, never from `@lyx_unseen`, `@lyx_hidden` or a wait mark.
- `PATTERN-agent-name`: the driver role is matched only through `internal/agentname`.
- `PATTERN-test-speed` and `PATTERN-test-economy`: the rule is pinned by a unit table, and the tmux-tier focus subtests are extended rather than duplicated.
- The doc paragraph in `internal/reedengine/doc.go` that names every automatic switch is updated in the same commit.

## Auto-mode assumptions

- "The Driver window" is the window of the strand whose role is `driver`, the role `loomengine` names its driver strand by.
- A failed read of the current window or the session's panes keeps the viewer where it is, as today.

## Open risks

- A replacement of the viewed agent now lands the viewer on the replacement's window when the agent the removal fell back to is the newest live one; the tmux-tier replace subtest's expectation changes with it.

## Acceptance criteria

- A new agent window takes the viewer from the newest live agent window.
- A new agent window takes the viewer from the Driver window while a newer live agent exists.
- A new agent window keeps a viewer on an older live agent window that is not the Driver's, and on the shell.
- The overview, all-dead agent window and Focus cases behave as before.
- `lyx gate test ./internal/reedengine` and `lyx gate test --tags tmux ./internal/reedengine` pass.

## Notes for the plan writer

- `newAgentWindowTakesViewerLocked` already lists the session's panes; it also needs the session's windows for `relevantWindow`.
- The tmux-tier subtests live in `internal/reedengine/windoworder_integration_test.go`.
