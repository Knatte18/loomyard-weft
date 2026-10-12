---
round: 1
focus:
  - "Local-layer rule (cards 2, 13, 14, 17, 20, 21): check that every entry point (loaders, `lyx config --layer local`, `configreg.PrepareRunSettings`, board write, Seed-Child) goes through `configengine.ValidateLocal`, and that `LocalKeys` and `LocalDenied` derive from the same classification the loaders use."
  - "Transitional loader geometry (cards 2-12, 15): check that each module loader's move from `Dirs{Anchor: baseDir}` or `Dirs{Board: baseDir}` to told dirs preserves behaviour, and that no strict loom load runs on a hub without a base layer between cards 9 and 15."
  - "Per-key classification (card 13): check that the `ConfigSpec().Categories` of every module match the plan's per-key table, and that the pin test fails on an unclassified template key."
  - "Reconcile and migration (cards 15, 16): check that the base layer stays full and upper layers sparse, that per-worktree repo defaults are seeded by the walk a clone also runs, and that the one-time strip is marker-guarded and idempotent."
  - "Seed-Child write (cards 19-21): check that run settings reach the pair's untracked `.lyx/config/<m>.local.yaml` only, never a tracked file, and that the task-slug guard is the shared one."
  - "Test economy: check the new tests per package against existing coverage, and the `cmd/lyx/testdata/test-budget.yaml` rows."
---

Seed focus drawn from the plan's shared decisions and card index; the reviewer settles each against the committed diff.
