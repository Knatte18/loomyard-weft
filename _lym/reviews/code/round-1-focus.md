---
round: 1
focus:
  - "internal/loomcli/halt.go `failedHalt`: does `errors.As(err, &skew)` see `*stencilstore.NewerBoardCopyError` on the real path from a seat's stencil render to the error the step returns, or does a wrap on that path use `%v`, a string or a non-wrapping type that hides it?"
  - "Every caller and consumer of `loomengine.ClassifyHalt` and of the anomaly kind set: does any other caller, doc, spec, selfreport intake filter or kind list need `build-skew`, and does the diff land it where the repository's rules require?"
  - "The decision record's added entry asks that a hand-over stop is not filed as `producer-hard-failure` and gets no issue from the selfreport path: does the diff cover each place that note reaches an issue, not only the reflection stencil?"
  - "`ClassifyHalt` case order and its doc comment: an interrupted or blocked halt carrying a build skew, and the claim that at most one kind fires."
  - "Stencil wording in `friction-template-reflection.md`: does the new `build-skew` rule keep every earlier drop and file rule intact, and is its stated reason true for a run whose driver has no newer on-disk build?"
  - "Tests in `halt_test.go` and `anomaly_test.go`: do the changed tests still check what their names say, was any assertion widened or removed, and do the new cases cover the build-skew path end to end from an error value, not from a preset flag?"
  - "The commit's `Root-Cause:` trailer: does it name the defect the diff removes, and does the commit message make any claim that needs before and after figures?"
---

Seed focus from the decision record's Option (b) and the six-file diff.
No change description was read, so the merge-danger and summary checks are left to the reviewer.
