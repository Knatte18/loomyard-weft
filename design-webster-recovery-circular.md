# webster recovery way-forward is circular: recover-batch points to accept-audit, which refuses; reset is refused while run.lock is held

`webster-recovery-circular` · Note · bug, webster

Imported from [issue #878](https://github.com/Knatte18/loomyard/issues/878).

## What happened

After `lyx webster record-batch` failed a batch on fabric-reference audit findings, `lyx webster recover-batch` refused with `needs_fresh` and a way forward of "`lyx webster accept-audit --batch 6` then `lyx webster recover-batch 6`", or `reset --to batch-start`, or `reset --to start`.
`lyx webster accept-audit --batch=6` then refused, because the batch's commits are kept only when every fabric reference is read-only, and the finding is the very one that made recover-batch advise it. Its only way forward was `lyx webster reset --to batch-start --batch 06`.
`lyx webster reset --to=batch-start --batch=6` was refused with `run is already in progress ... (run.lock held)`, with way forward "wait for it to finish, or check `lyx webster status`".
The Webster session had no verb left that it was allowed to use, so it escalated: the run halted `blocked` with anomaly `escalation-to-human`, and the halt note says the parent has to run the reset after the run stops.

## Why it is abnormal

- `recover-batch` offers `accept-audit` as a way forward in a case where `accept-audit` is known to refuse; the advice cannot be followed.
- The refusals were retried several times (two `accept-audit` refusals, three `recover-batch` refusals), each with the same text.
- `reset` is refused for a lock the Webster session itself holds, so the one working remedy is unreachable from inside the run that needs it.

## What to do

- Have `recover-batch` run the `accept-audit` check and drop that option from the way forward when it would refuse.
- Let `reset --to batch-start` run from the Webster session that owns `run.lock`, or have the refusal name who must run it and when.

## Provenance

Task `umbrella-anchor`; notes: webster-refusal-recover-batch.md, webster-refusal-recover-batch-2.md, webster-refusal-recover-batch-3.md, webster-refusal-accept-audit.md, webster-refusal-accept-audit-2.md, webster-refusal-reset.md, loom-halt-3.md.
