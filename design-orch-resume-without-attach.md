# The orch resumes runs without commands that take over the operator's terminal

`orch-resume-without-attach` · Note · bug, orch, loom, reed

Seen 2026-10-06: the prime's reed window jumped to other sessions "by itself" while the operator watched the orch.
The cause was the orch's own `lyx loom start` calls in task worktrees: the orch runs inside reed's tmux, so `$TMUX` is set, and `lyx loom start` without `--no-attach` runs `tmux switch-client` onto the task's session (`internal/loomcli/start.go`), taking the operator's terminal with it.
`lyx orch start` does the same without `--no-attach` (`internal/orchcli/handover.go`).

The orch role (`contracts/stencils/orch/orch-template-role.md`) tells the orch to resume a halted child with `lyx loom start`, an operator verb whose job includes handing over the terminal.
In practice the call also did not move the runs: a live driver resumed only once the orch messaged it.

## Direction

- The orch resumes a child through a verb with no interactive side effect: a resume verb that only records the decision and wakes the run's driver (or batten), never spawning or attaching anything.
  The role stencil names that verb and never `lyx loom start`.
- `lyx loom start` and `lyx orch start` never switch the tmux client when called from an agent (`LYX_STRAND_NAME` set), whatever the flags; the terminal handoff is for an operator's shell only.

The bug half (no tmux switch-client from an agent) is in task `bugfix-sessions`; this note keeps the orch resume verb.

Operator decision 2026-10-06: start verbs never attach a terminal for any caller; attaching is only `lyx reed attach` (implemented in `bugfix-sessions`).
