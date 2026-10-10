# webster: a second recovery's prompt marks the first recovery's uncommitted files as not written by this run

`recovery-attribution` · Note · bug, webster

Imported from [issue #522](https://github.com/Knatte18/loomyard/issues/522).

Seen in run quick-recipe (2026-10-10): batch 01's first recovery strand (shuttle run 0316a099) edited 19 files for card 1 and was stopped uncommitted when Webster restarted. The second recovery's prompt listed all 19 under "Not written by this run: leave it untouched, never stage or commit it", so the strand blocked on card 1.

Workaround: orch told the recovery strand to treat the 19 files as batch 01's own work and commit them under card 1.

Expected: files an earlier recovery attempt of the same batch wrote count as the batch's own work (attribution by batch, not by shuttle run id), or the second recovery starts from a clean tree with the first attempt's diff offered as reference. Related: recover-batch 1 exited 0 while the batch ended failed and terminal.