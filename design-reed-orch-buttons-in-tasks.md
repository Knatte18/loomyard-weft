# reed: the orch's slack and RC buttons show in task sessions

`reed-orch-buttons-in-tasks` · Note · bug, reed, orch, slack

## Seen (operator, 2026-11-17)

The slack and RC buttons in reed's status bar belong to an orch, one state per orch:
the prime's orch and each beam's orch has its own (`lym orch rc-bar` and `lym slack bar` per worktree).
They also show in a task's own tmux session, where the operator reads them as the task agent's buttons
and wonders whether RC is all orchs or none.

They show there because a task session's `@lym_button_dir` names its parent's worktree, the prime's or the beam's,
so the state jobs in `@lym_rc_state_command` and `@lym_slack_state_command` read and switch the parent orch.
A click in a task session turns on the parent orch's remote control or Slack, never the task's.
`internal/reedengine/bar.go` says the slack toggle is "drawn only in a prime's or a beam's session", which the code does not hold.

## Idea

- Draw the slack and RC buttons only in the session of the worktree whose orch they switch, the prime's or a beam's,
  and none in a task session.
- Or, if they should stay reachable from a task, label them with the orch they switch, such as `RC prime` or `RC <beam>`.
- Make the bar's doc comment match what is drawn.
