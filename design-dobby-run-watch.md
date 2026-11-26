# Built-in run watch: mail the parent on every non-running state and on a gate slot held too long

`dobby-run-watch` · Task · feature, dobby, gate · starting · Worktree-Create

Imported from [issue #222](https://github.com/Knatte18/loomyard/issues/222).

## What the orch does by hand today

In hub `dop`, the operator requires the orch to hear of every stop of every run it parents.
lym notifies some halts, but not all: a Webster block sends no notice (#207), and a gate whose testhost hangs past `timeout_min` holds its slot with no notice at all (#216).

So the orch keeps a Claude Code Monitor per run in flight: a shell loop that polls `lym loom status` in the task worktree every 30 s, prints on every change of state or producer, and prints a warning when `lym gate status` shows the run holding a gate slot past 45 minutes.
The Monitor expires after 30 minutes, so the orch re-arms it by hand, and a missed re-arm left a hung tier3 gate unseen for 21 hours.

## Wanted

lym itself watches every run in flight and mails its parent (creel) on:

- every state other than running: blocked, failed, paused, done, whatever the cause;
- a gate slot held by the run past a bound, such as 1.5 × the tier's `timeout_min`, naming the gate's PID and its child processes;
- optionally, every row change, for parents that want progress.

The watch lives in the dobby, not in the orch's session, so it needs no re-arming and survives the orch's context cycles.
