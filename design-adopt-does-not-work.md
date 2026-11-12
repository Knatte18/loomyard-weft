# lym adopt does not work

`adopt-does-not-work` · Note · bug, orch, fabric

## Seen (operator, 2026-11-12)

The operator says `lym adopt` does not work today; no error text was given with the report.
The last adopt record in Bolt (`.lym/adopt/adopt.json`) is from 2026-11-11 14:37 and its root check was green, so the failure is newer or not in that record.

## Idea

Reproduce it first: run `lym adopt --here` (or the subject that failed), keep the output and the adopt record, and find which step stops (reconcile, pairs, pause, owner, deploy, beam, check, done).
Then fix it as a darn task.
The re-parent verb (`reparent-running-task` report, `docs/research/loom/reparent-running-task-2026-11-12.md`) waits until adopt works.
