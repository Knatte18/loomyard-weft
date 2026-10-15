---
round: 1
focus:
  - "internal/planparser/edittestfile.go and its wiring in internal/planglyph/index.go and internal/loomcli/validate.go: settle whether the edit-test-file check is appended at both the format gate and the rework gate, and whether it matches the loom-plan-spec row card 1 adds."
  - "internal/planglyph/callercoverage.go and plangate.go: settle whether the caller-uncovered and redundant-file-target ways forward can still name a _test.go file in any branch, including the redundant-file-target path."
  - "contracts/stencils/loom/loom-template-plan.md and loom-rubric-plan-review.md: settle whether the stencil still shows a plain test-file path under Edit anywhere, and whether the Uses-neighbours rule matches the plan's wording."
  - "Test economy across edittestfile_test.go, index_test.go, parity_test.go and glyphchain_test.go: settle whether any new test repeats coverage an existing test already gives, and whether cmd/lyx/testdata/test-budget.yaml was raised only as far as the new top-level tests require."
  - "doc.go and spec updates (internal/planparser/doc.go, internal/planglyph/doc.go, contracts/specs/loom-plan-spec.md): settle whether the documented check list and ordering match the code."
---

Seed focus only; each entry names where to look, and the reviewer judges the diff on its own.
