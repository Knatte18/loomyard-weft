---
verdict: APPROVED
---

No findings.

## Rejected

- `TestLoomRubrics`' new `namesSpecs` assertion fails for one of the five rubrics: rejected.
  The assertion is that the rendered text contains `/abs/specs/dir` exactly when `namesSpecs` is set.
  `{{.specs_dir}}` appears in `loom-rubric-discussion-review.md`, `loom-rubric-plan-review.md`, `loom-rubric-webster-review.md` and `loom-rubric-merge-review.md`, the four rows set true, and `internal/shedadapters.ReadRubric` fills `specs_dir` verbatim from its argument.
  `loom-rubric-darn-review.md` carries no `{{.specs_dir}}`, so its false row holds.
  Card 8 adds no marker to any rubric: its darn-review bullets name the board entry and the Test economy items, none a spec citation.
  The assertion holds for every row before and after card 8.
- The `//testtiming:keep` reason rewrite attaches to a different test than the card says: rejected.
  Of the two files the focus names, only `contracts/stencils/markers_test.go` carries the line, directly above `TestStencils_TopLevelMarkers`, and the card's rewrite names that test.
  `contracts/stencils/rubric_test.go` has no `//testtiming:keep` line, and the card asks for no rewrite there, only for its file comment to drop the `claims_test.go` line.
  The two `//testtiming:keep` lines in `claims_test.go` leave with the file.
- A kept row in card 1's three tests departs from D2: rejected.
  The `gatecli.`, `shedrun.` and `shuttleengine.` rows of `claims_test.go` are the `webster-body-implementer` slot-busy exit, the three `MessageTail` stencils and the shed driver's park marker, matching the card's `TestStencils_NameGoValues` rows, and the `status:` and `head_sha:` keys are the `yaml` tags of `websterengine.Report`.
  The section-scoped marker rows' headings exist in `loom-template-plan.md`, `loom-template-discussion.md` and `shed-template-driver.md`.
  The package's top-level test count stays at 16 against the budget row: three leave, three arrive.
