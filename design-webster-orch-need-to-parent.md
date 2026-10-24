# Webster batch whose card verify needs an orch-only change dead-ends: recover-batch ends dead with 'asking', then the run halts repeatedly with growing history

`webster-orch-need-to-parent` · Task · bug, webster, loom · done

Imported from [issue #867](https://github.com/Knatte18/loomyard/issues/867).

What happened: in batch 22 the fork reported stuck because a card's verify needed the hub's tier commands changed, which only the orch can do.
`lyx webster recover-batch 22` ended dead with dead_reason asking and recovery_retry false, and the refusal said to end the run stuck.
A second refusal record for the same batch shows dead_reason died, recoveries 1 and recovery_retry true, with the way forward 'run recover-batch 22 once more'.
The run then halted blocked (anomaly escalation-to-human) three times; each re-entry appended further Webster stuck entries to the history, and the later halts said the recovery was exhausted and that later batches were not begun.
Separately, a begin-batch refusal reported plan_conflict tree_moved for a card target whose unit exists but whose member is missing, with the way forward being rebaseline.

Why abnormal: a need that only the orch can satisfy is classified as a dead recovery instead of a request to the parent, so the recovery budget is spent and the run can only halt for a human; the two refusals for one batch give contradictory recovery_retry and dead_reason values, and repeated re-entry of a halted run adds stuck history without progress.

What to do: let a fork's 'needs orch action' report route to the parent (a message or a distinct pause state) without consuming the recovery budget, keep recovery_retry and dead_reason consistent across consecutive recover-batch refusals, and make a re-entry of an already-halted escalation return the existing halt rather than append another stuck entry.

Provenance: task gate-runs-once; notes loom-halt.md, loom-halt-2.md, loom-halt-3.md, webster-refusal-recover-batch.md, webster-refusal-recover-batch-2.md, webster-refusal-begin-batch.md

## Design choice (operator left it to the orch, 2026-10-24)

A fork's "needs orch action" report puts Webster into a waiting state of its own that spends no recovery budget, and the orch is told; once the orch has acted, the batch resumes where it stood.
No distinct halt type is added.