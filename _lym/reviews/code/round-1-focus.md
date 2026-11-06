---
round: 1
focus:
  - "internal/websterengine/beginbatch.go, BeginBatch: does `orderOnly` become true only when every blocking finding is delete-before-reference, and can it stay true with zero blocking findings or be wrongly set by an informational finding?"
  - "internal/websterengine/beginbatch.go, PlanConflictError.Error: does the plan_order way forward keep the rebaseline and begin-batch tail correct, and does the fix-plan path (planfixer, webstercli fixplan) still accept a plan_order cause where it handled plan_defect and tree_moved?"
  - "Other readers of the conflict cause (internal/websterengine/planfixer*, internal/webstercli, internal/shedadapters, the webster master stencil): does any switch, validation or doc still name only the two old causes, so plan_order is rejected or misrouted?"
  - "internal/planglyph/deleteorder.go: do all three delete-before-reference emit sites use planindex.CheckDeleteBeforeReference, and does any other emitter or string comparison of that check name remain as a literal that could drift?"
  - "Answer key (board entry, issue #68): the entry asks to classify the conflict as a plan-ordering one and to consider running the check at plan accept or rebaseline; does the diff and its spec/doc prose about that second item match what the code does?"
  - "internal/websterengine/beginbatch_test.go: does the new test cover behavior not already covered by the existing begin-batch conflict tests, and does it cover each branch of the cause switch?"
---

Seed focus for round 1. The diff is a single commit; the decision record carries only the imported board entry under `## Goal`.
