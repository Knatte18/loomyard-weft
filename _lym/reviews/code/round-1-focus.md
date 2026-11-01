---
round: 1
focus:
  - "internal/loomshed/seed.go and loomshed.go: does the seed still name any producer, and does every caller (tapestry, darn, pawl bootstrap) get the first recipe row as the start without a hard-coded name?"
  - "internal/shedengine/run.go, status.go, progress.go: how does the status file record the current producer before the first pass, and does a run with no named producer start at the recipe's first row for every recipe?"
  - "The verbs that move a run (goto, resume) in shedengine and loomengine/anomaly.go: does each behave correctly when the status carries no producer name, as the decision record's Design asks the task to check?"
  - "internal/loomrecipe/recipe_test.go removes 31 lines: does the deletion drop coverage of behavior that still exists, and is the new run_routing_test.go the covering test?"
  - "contracts/specs/loom-status-spec.md and shedengine/doc.go: do the doc changes match the code's seeded-status behavior, and are the docs the repository's rules name for this kind of change present?"
---

The decision record holds a board entry only, so the amended entry's Design is the measuring stick.
