```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List per behaviour the test that pins it, or none.

Per-behaviour list:
- Discussion spec rule (`contracts/specs/loom-discussion-spec.md:88-90`, `Review:` prefix): static spec prose. No Go code changed, so exempt from the coverage rule. The decision-id check (`internal/discussionparser/decisionids.go`, `CheckDecisionIDs`) matches `D<n>` as a whole word anywhere in a bullet. It is not changed by the diff, and a `Review:` prefix does not change what it matches. Pinning test: none, and none is needed, since no behaviour changed.
- Discussion stencil Testing item (`loom-template-discussion.md:88-89`): static stencil prose. Exempt. No marker, Go-filled value or fragment changed, so the marker tests in `contracts/stencils/markers_test.go` are unaffected.
- Plan rubric Test economy bullet (`loom-rubric-plan-review.md:59-65`): static rubric prose. Exempt. `contracts/stencils/rubric_test.go` pins only markers and the review base. No rubric marker was added or removed.
- Code rubric Wording criteria bullet (`loom-rubric-code-review.md:100-103`): static prose. Exempt, for the same reason.
- Tests added by the diff: none. The diff changes no `.go` file, and it adds no assertion on prose. That matches the amended entry and `pattern/PATTERN-test-speed.md` ("No test pins a stencil's static prose").

Findings: none.

I made no tool calls. The inherited context already held the diff, `decisionids.go`, `rubric_test.go`, `markers_test.go` and `PATTERN-test-speed.md`.
