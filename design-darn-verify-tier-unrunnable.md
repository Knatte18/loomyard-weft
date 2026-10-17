# A darn writer cannot rerun a verify tier it is refused, so a flaky tier failure loops and escalates

`darn-verify-tier-unrunnable` · Note · enhancement

Imported from [issue #714](https://github.com/Knatte18/loomyard/issues/714).

## What happened

`lyx loom` halted `blocked` (anomaly `escalation-to-human`) at Publish.
`publish_verify` failed after merging the parent branch with `internal/reedengine` `TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` reporting `run list-commands: exit status 1: server exited unexpectedly`.
The task's diff did not touch `internal/reedengine`, so the failure looks like a race in booting two tmux servers concurrently, not a defect of the change.

## Why it is abnormal

- The test is flaky and fails an unrelated task at the Publish gate.
- The Darn-Write agent could not reproduce or fix it: the test carries the `tmux` build tag, and tmux-tier go runs are refused in a darn session.
  The agent handed off with no commit, so the run escalated to a human with a failure the darn step can never resolve.

## What to do

- Make `TwoWorktreesBootedConcurrently` deterministic, or retry or isolate the tmux server start it races on.
- Consider whether a darn session facing a verify failure only in a tier it cannot run should get a way to rerun it, or should be told up front to escalate, rather than looping through the Darn cycle first.

Provenance: task `stencilstore-dev-warning-noise`; notes `Darn-Write.md`, `loom-halt.md`.
