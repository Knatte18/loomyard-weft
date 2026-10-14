---
round: 1
focus:
  - "cmd/lyx/testdata/test-budget.yaml: both sides edited different rows and merged cleanly; check that every package the task added top-level tests to (planparser, batcher, websterengine, impactset, contracts/stencils) has a row at or above its merged count, and that the parent's removed pausecli/pauseengine rows and added tendercli/tenderengine rows leave no stale or missing row."
  - "contracts/specs/refusal-spec.md: the task added a webster-section row, the parent rewrote the pause-section rows for tender revive; check the merged file keeps the table format the spec contract requires and that no row of one side contradicts the other."
  - "Parent's rename of pausecli/pauseengine to tendercli/tenderengine: check whether any code, test, stencil, doc or guard the task added still names the old packages or the old pause/resume wiring."
  - "pattern/PATTERN-cli-cobra.md deviation list changed by the parent: check the task's code (new CLI surface or verbs, if any) follows the changed line."
  - "Task's guard-test scan in websterengine/impactset/planparser against the parent's new packages and files (tenderengine, shuttleengine/conditionedsend.go, proc, reedcli): check the new guard walk and CardCovers/boundary logic handle the parent's added files, and that its new tests do not duplicate parent-added coverage."
---

Seeded from the overlap list and the two sides' file lists; no round has been judged.
