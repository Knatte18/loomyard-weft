# The background push of a cancel deletes the weft branch of a new run of the same slug

`cancel-push-deletes-new-weft` · Note · bug, batten, fabric

## Seen (2026-11-03)

`lym batten cancel shed-sub-run-block --remove` reported `push_spawned: true`, and the task was released again within a minute.
Dobby started a new run of the same slug, which made a new pair and pushed its weft branch.
At Seed-Child the new run failed at bootstrap: origin no longer had branch `shed-sub-run-block-weft`, so the cancel's background push most likely deleted the remote branch after the new run had pushed it.
The tackler handed the stop back; the orch removed the pair with `lym fabric remove` and released the task again.

The way forward the stop named, `lym fabric remove`, removed the pair but left the failed run's record: the hub's run overview showed two runs of the task, the failed one and the ready entry, and dobby kept failing the old run's batten pass with "no pair worktree".
A further `lym batten cancel` removed the record; origin still held the run's warp and weft branches, which the orch deleted by hand (the weft head is kept by its archive tag) before releasing the task again.

## Expected

A cancel's background remote work never removes a branch that a later run of the same slug owns: the new run waits until the cancel's push has finished, or the push deletes the remote branch only when it still points at the commit the cancelled run left.
