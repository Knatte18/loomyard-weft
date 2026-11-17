# reed: an RC button per agent, and Slack only in the orch's session

`reed-orch-buttons-in-tasks` · Task · bug, reed, orch, slack · running · Drift-Verify

## Seen (operator, 2026-11-17)

The slack and RC buttons in reed's status bar belong to an orch, one state per orch:
the prime's orch and each beam's orch has its own (`lym orch rc-bar` and `lym slack bar` per worktree).
They also show in a task's own tmux session, where the operator read them as the task agent's buttons.

They show there because a task session's `@lym_button_dir` names its parent's worktree, the prime's or the beam's,
so the state jobs in `@lym_rc_state_command` and `@lym_slack_state_command` read and switch the parent orch.
A click in a task session turns on the parent orch's remote control or Slack, never the task's.
`internal/reedengine/bar.go` says the slack toggle is "drawn only in a prime's or a beam's session", which the code does not hold.

A click on RC already turns remote control on in a running session:
`turnOnRemoteControl` (`internal/orchcli/remotecontrol.go`) queues creel's remote-control command into the orch strand,
and records the strand's state for the button.
Nothing in that path needs the strand to be an orch.

## Decided (operator, 2026-11-17)

- Slack stays per orch: its buttons show only in the session of the prime or the beam whose orch they switch, none in a task session.
- RC is offered for every agent, not only orchs: the RC button belongs to the agent of the window's active pane,
  and a click turns remote control on for that agent alone, through the same creel command.
- RC is off for every agent until its button is clicked.
  No repository switch turns it on for agents; `orch.yaml`'s `remote_control` for orch sessions stays as it is.
- The bar's doc comment matches what is drawn.
