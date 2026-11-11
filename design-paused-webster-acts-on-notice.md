# A paused Webster session acts on a decision notice without the run lock, and the loom row stays blocked

`paused-webster-acts-on-notice` · Note · bug, webster, loom, creel

Imported from [issue #101](https://github.com/Knatte18/loomyard/issues/101).

## What happened

A tapestry run's Webster paused on the parent's order: Webster wrote `outcome: stuck`, the loom row went to `blocked · Webster`, and the outer batten driver yielded.
The Webster session itself stayed open: its strand (`<hub>:<task>:webster`) was still live in `lym reed list`.

Later the parent recorded a decision on the run with `lym loom decision add <slug> --by parent`.
That verb notified the live Webster strand (`"notified":["<hub>:<task>:webster"]`).
The idle Webster session woke on that notice and went on working in the old session, outside any `lym webster run`:
it forked the batch, merged, committed two cards, ran the verify gate and wrote `outcome.yaml` with `outcome: done`.
`record-batch` warned that no live `lym webster run` owned the state, because the session was not started under the run's lock.

## Result

`lym webster status` showed every batch done, but the loom row stayed `blocked` at Webster with the old pause reason, and `lym batten status` showed `failed` with `battencli: cannot resume`.
`lym batten run <slug>` yielded with the stale blocked error.
The overview page showed the tapestry as blocked although its work was done.
Running `lym loom resume` in the task worktree was needed to get the driver to re-run Webster and move on.
Meanwhile the parent's own `lym webster run` would have started a second Webster beside the lockless one.

## Expected

A Webster that reports stuck ends its session, or a session that does not hold the run's lock refuses to act on mail and says so.
`lym loom decision add` does not wake a producer strand whose run is halted, or warns the parent that it did.
`record-batch` without the lock refuses rather than warns, so the work and the loom row cannot drift apart.
