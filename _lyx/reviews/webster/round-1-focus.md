---
round: 1
focus:
  - "Stencil wording: every changed stencil (plan, Plan-Review rubric, implementer, driver, advisor, rework) against its card's Intent. Settle whether a rule the card did not ask to change was dropped, weakened or contradicted."
  - "Shared Decision 'One marker value': every site fills the `verify_tiers` marker through `languageconfig.VerifyTiersValue`, the marker is required wherever it appears, and every fallback sentence lives in the stencil, not in Go. Settle whether any site builds the value or its surrounding sentence another way."
  - "Per-card mechanical check: confirm each card's own groups ran their type-specific check (Rename, Delete and the like) against that group's own targets. Check the renders in `internal/webstercli`, `internal/websterengine`, `internal/shedadapters` and `internal/loomengine` for a missed site."
  - "Test economy: new tests in `internal/loomengine/plan_test.go`, `internal/websterengine/template_test.go`, `internal/shedadapters/rubric_test.go` and `internal/loomcli/wiring_test.go`. Settle whether any duplicates an existing test, asserts a stencil's static prose, or leaves a behavior the diff adds uncovered."
  - "Comment-convention compliance for new or changed doc comments, for instance `internal/websterengine/doc.go` and the `internal/languageconfig` change, against PATTERN.md and the surrounding code."
---

Seed focus from the plan's Card Index and the diff stat.
