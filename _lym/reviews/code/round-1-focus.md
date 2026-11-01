---
round: 1
focus:
  - "Cut content in `contracts/specs/loom-plan-spec.md` (diff vs. base): does any rule a plan writer or reviewer acts on, or any check's fix, vanish rather than move to `loom-plan-checks-spec.md`, `loom-plan-csharp-spec.md` or `loom-parts-spec.md`?"
  - "Every reference to the old spec sections or anchors (stencils under `contracts/stencils/loom` and `contracts/stencils/webster`, `internal/planparser/validate.go`, `internal/webstercli/validate.go`, `internal/planglyph/doc.go`, `docs/overview.md`, the other loom rubrics and specs): does any still point at a heading, file or section name that no longer exists after the split?"
  - "`contracts/specs/specs.go` and `specs_test.go`: are the new files registered, embedded and resolvable the same way the existing spec is, and does each role that read the old spec now read every part it needs?"
  - "Stencil edits (`loom-block-plan-cards.md`, `loom-block-plan-format.md`, `loom-block-plan-overview.md`, `loom-rubric-plan-review.md`, `webster-template-plan-fix.md`): is any rule dropped, weakened or contradicted that the decision record's Goal did not ask to change?"
  - "`internal/planparser/validate_test.go` change: does it still assert what the test did before, and does it add no redundant coverage?"
  - "Commit messages and `_lym/landing/summary.md` claims (if present): does the diff bear out each claim, such as size reductions and what moved where?"
---

Seed focus from the Goal in the decision record and the diff stat of the review range; the record has no other sections to check.
