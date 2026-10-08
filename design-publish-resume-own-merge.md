# Loom resume at Publish refuses the dirty worktree left by its own staged merge-in, so the run cannot recover

`publish-resume-own-merge` · Note · bug, loom, fabric

Imported from [issue #471](https://github.com/Knatte18/loomyard/issues/471).

## What happened

After a `Publish` step failed mid merge-in (merge staged, conflict resolved by the agent, then a producer refusal), the loom driver re-stepped once.
The run then halted as `blocked` at `Publish`, anomaly `escalation-to-human`, with this reason:

`the task worktree has uncommitted changes before the merge-in: <a long list of files, plus "and 63 more">; commit or remove them on the task branch, then resume`

The listed files are the changes the staged merge-in itself brought in, not the task's own uncommitted work.

## Why it is abnormal

The merge-in precondition check treats the worktree state produced by the earlier, interrupted merge-in attempt as foreign uncommitted changes.
A re-step or resume after a partial merge-in can therefore never pass the check, and the run escalates to a human although all earlier segments (Discussion, Plan, Webster, Describe) were `done`.
The advice in the message ("commit or remove them") would either commit an unfinished merge or discard the resolution.

## What to do

- Detect an in-progress merge (`MERGE_HEAD` present, or merge-staged state recorded in the trace/history) in the `Publish` precondition and resume it, instead of applying the clean-worktree check.
- Alternatively provide a `lyx` repair verb that completes or aborts a staged merge-in, and name it in this refusal message.
- Make the loom driver's repair step aware of this state, so it does not re-step into a guaranteed refusal.

Related: the transient `object not found` report, which is what first interrupted the merge-in.

Provenance: task `binary-change-and-rules`, from `loom-halt.md` and `driver-repair-1.md`.
