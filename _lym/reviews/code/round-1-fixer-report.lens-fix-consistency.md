```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F3 change set against its surroundings.

no findings

- **F1:** the new subtest agrees with the `decisionRuleMarker` doc, the `FirstRaisedGates` field comment and `internal/shedadapters/doc.go:160-162`.
- **F2:** the rewritten stencil comment agrees with those same three. Line 46 of the stencil ("from round 2 on, where the decision rule says so") still holds.
- **F3:** outside `docs/research`, the search found no remaining `FocusFindingAdmitted`, `FocusFindingRefused` or `focusReview`. The "held focus finding" at `contracts/stencils/harden/harden-rubric-review.md:58` is about carried findings, not the removed exemption.

Tool calls:
1. `git grep` for the old names, the old rule's wording and `FirstRaisedGates`: my context did not show repo-wide twins. The output was too large and only its preview was read.
2. The same search narrowed to exclude `docs/research`: needed because the first output was truncated.
