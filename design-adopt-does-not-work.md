# lym adopt does not work

`adopt-does-not-work` · Note · bug, orch, fabric

## Seen (operator, 2026-11-12)

The operator says `lym adopt` does not work today, and that it is a lot of trouble: much housekeeping around every use.
No error text was given with the report.
The last adopt record in Bolt (`.lym/adopt/adopt.json`) is from 2026-11-11 14:37 and its root check was green, so the failure is newer or not in that record.

## Idea

Find out which part is the trouble before fixing anything.
Run `lym adopt --here` (or the subject that failed), keep the output and the adopt record, and note which step stops or needs hand work (reconcile, pairs, pause, owner, deploy, beam, check, done).
Then decide with the operator whether to repair the failing step or to cut the verb down to fewer steps.
The re-parent verb (report `docs/research/loom/reparent-running-task-2026-11-12.md`) waits until adopt is settled.
