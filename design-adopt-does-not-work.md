# lym adopt does not work

`adopt-does-not-work` · Note · bug, orch, fabric

## Seen (operator, 2026-11-12 and 2026-11-15)

The operator says `lym adopt` almost never works, and that it is chaos: much housekeeping around every use.
No error text was given with the report.
The last adopt record in Bolt (`.lym/adopt/adopt.json`) is from 2026-11-11 14:37 and its root check was green, so the failure is newer or not in that record.
On 2026-11-15 the hub came back from OSL-1046 to Neve; the root adopt check was green on tiers 0, 1 and 2,
but the orch came up as a new session (note `orch-resume-after-owner-move`) and finished bench pairs still stood (note `stale-bench-pairs-linger`).

## Idea

Find out which part is the trouble before fixing anything.
Run `lym adopt --here` (or the subject that failed), keep the output and the adopt record, and note which step stops or needs hand work (reconcile, pairs, pause, owner, deploy, beam, check, done).
Then decide with the operator whether to repair the failing step or to cut the verb down to fewer steps.
The operator's lasting fix is the note `lym-state-fsm-in-git`: with every run's state an FSM pushed to GitHub, adopt becomes pull and go on.
The re-parent verb (report `docs/research/loom/reparent-running-task-2026-11-12.md`) waits until adopt is settled.
