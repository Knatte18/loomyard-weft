# Recovering batch 01: the earlier recovery attempt's work was labeled "Not written by this run"

The recovery prompt listed all 19 uncommitted card-1 files under "Not written by this run: leave it untouched, and never stage or commit it".
They were the unfinished card-1 work of the previous recovery strand for this same batch (shuttle run `0316a099…`), which was stopped when Webster restarted before it committed.
The attribution seems to go by shuttle run id, so a second recovery attempt sees the first attempt's work as foreign.
Following the rule literally made card 1 impossible, so I had to stop and ask `ly:orch`, which confirmed the files were batch 01's work.

The same prompt also left it unclear whether the prior rejection ("never run a fabric-referencing command") forbade the `lyx gate` and `lyx loom lint-comments` gates, and card 3's tmux-tagged scenario steps can only be compile-checked here because no allowed gate runs the tmux tier.
