---
round: 2
focus:
  - "internal/shedverbs/step.go `shortStepEnvelope` and `shortStepErrFields`: does a test in `internal/shedverbs` run the short envelope with a non-empty friction on success, a non-empty friction on refusal, and a producer `ParentNotice` on success, and assert the short key set then carries `friction` and `parent_notice`?"
  - "internal/shedverbs/steprecord_test.go: is the file gofmt-clean, and does any test added or changed in the range still format correctly?"
  - "internal/shedrun/paths.go `ParkMarker` doc comment: does it name the full step envelope held by the step record, not the printed short envelope, as the source of `scratch_dir`, with one sentence per line?"
---

The focus entries target the three round-1 sites a fix touches; the rest of the range is reviewed as usual.
