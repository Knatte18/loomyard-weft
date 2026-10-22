---
round: 1
focus:
  - "contracts/stencils/loom/loom-rubric-plan-review.md: the parent added a bullet about a card that carries a research step, and the task edited the same file and `loom-template-plan.md`. Settle whether the merged rubric and the merged plan template still agree on what a card may leave to be found out while it runs."
  - "PATTERN.md: both sides edited entry lines (parent: sparse-hub-base, config-layers, new template-overlay; task: glyph-conversion-chokepoint, review-round, testkit). Settle whether the merged file keeps every entry's line consistent with its `pattern/` background file and whether the task's code follows the parent's new `PATTERN-template-overlay` and changed `PATTERN-config-layers` lines."
  - "internal/loomcli: the parent added darn classification, decision and landing-deps code, while the task changed `parity_test.go`. Settle whether the task's parity expectations still hold against the parent's new wiring, and whether the new parent code reaches a plan ref or glyph through anything but `planparser` and `glyph`, as the task's changed chokepoint line requires."
  - "Test economy on the interaction: check whether any task test in `internal/loomcli` or `internal/planparser` now duplicates a parent test in `darnclassify_test.go`, `decision_test.go` or `landingdeps_test.go`, and whether any behavior where the two sides meet has no covering test."
---

Seeded from the overlapping files and package in the merge note; the entries name where to look, not verdicts.
