# webster record-batch 01 fails on create-not-done for a card named 3-proc-group-kill

`record-batch-card-scope` · Note · bug, webster

Imported from [issue #484](https://github.com/Knatte18/loomyard/issues/484).

`lyx webster record-batch 01` refused with a failed batch.
The refusal names batch `01-headcontains-bounded-walk` but blames card `3-proc-group-kill`: `create-not-done/3-proc-group-kill[blocking]: Create target "plan:internal/proc#ConfigureGroupKill" still does not resolve`.
The report was archived under `_lyx/webster/reports/` and the way forward given was `lyx webster recover-batch 01`.

Why this looks abnormal: the failing card's slug differs from the batch being recorded, so either the create-not-done check evaluates cards outside the batch being recorded, or the batch and card numbering are mixed up.
It is also unclear whether the "still does not resolve" check resolves the `plan:` symbol against the right tree (the worktree head versus the card's merged result), since the symbol was later made to resolve by a follow-up commit.
If the check is correct and the card simply had not created the symbol, this is a task defect and can be closed.

What to do: confirm that `record-batch` only applies create-not-done to cards of the batch being recorded, and that the refusal message names the batch and card consistently.

Provenance: task `git-robustness`; notes: `webster-refusal-record-batch.md`.
