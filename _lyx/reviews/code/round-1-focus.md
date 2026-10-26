---
round: 1
focus:
  - "internal/planparser/validate.go: does `rename-leaves-test` match only Go member glyphs, derive the base `Name` or `T_M` through `ParseRef`, accept `Test<base>` and `Test<base>_<suffix>` but not `TestNewBeamer`, skip tests the plan renames, and stay silent under csharp and none?"
  - "internal/planglyph/drift.go and internal/planindex/planindex.go: does `RemovedTargets` exempt exactly an exact quarry rename, a report `renames` entry, and a Delete target or Rename old side of the batch's own cards, and cover a later card's `Uses:` refs as well as its targets?"
  - "internal/websterengine/recordbatch.go: does the refusal fire only for an OK fork report, once per batch, before BindHandles, leaving the batch non-terminal with report and bracket intact; does the second record-batch skip only this refusal; does recover-batch never refuse; is the new BatchState field read tolerantly?"
  - "internal/webstercli/recordbatch.go: does the envelope carry `later_target_removed` and the batch name, is the whole run state saved and fabric synced before the non-zero exit, and does the refusal name its way forward with a test reaching it?"
  - "Stencils and docs: do the Master rung and implementer body wording match the decision record's flow (SendMessage to the fork, record-batch again, unreachable fork) without dropping or weakening existing rules, and do `doc.go` files, the plan spec and the record-batch help text land the new behavior in the same diff?"
  - "Test economy in recoverbatch_test.go, recordbatch_git_test.go and the new rows: are the larger rewrites in recoverbatch_test.go needed by the change, and does each new test add coverage not already held by an existing one?"
---

Seed focus from the board entry and decision record; the diff touches planparser, planglyph, planindex, websterengine, webstercli and two stencils.
