# Webster run stopped at batch 13

This session resumed with batches 01 (payload-authority), 07 (attach-held-runs) and 11 (treadle-triage-retire) already recorded `done`, and batch 13 (tail-in-stencils-and-steer) recorded `failed`.
No batch was begun or forked in this session.

`lyx webster recover-batch 13` refused with `needs_fresh`.
The batch failed on a `fabric-reference` audit finding: the implementer fork ran shell commands from the hub directory that read `.lyx/shuttle/*/events.jsonl` files of other worktrees' fabric state.
Recovery cannot check that finding, so no recovery can clear it.

Batch 13 and every batch after it in the card list are therefore not built by this run.
The refusal names the way forward for the operator: when the batch made no commit and the worktree is clean, run `lyx webster accept-audit --batch 13` and then `lyx webster recover-batch 13`; otherwise run `lyx webster reset --to start` and then `lyx webster run --fresh`.
