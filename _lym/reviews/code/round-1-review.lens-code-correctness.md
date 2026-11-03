```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over a docs-only diff (four stencil/spec files, no Go code).

**List (function/branch → case run → verdict)**

- No Go function, branch or error path changes in the diff (`git diff --stat` shows only `contracts/specs/loom-discussion-spec.md`, `loom-rubric-code-review.md`, `loom-rubric-plan-review.md`, `loom-template-discussion.md`). Nothing to run step by step.
- Prose-as-rule branch that touches code: the spec's `Review:` prefix versus `CheckDecisionIDs` (`internal/discussionparser/decisionids.go:70-79`). Case: acceptance bullet `- Review: D3 the stencil names X`. `bulletLines` returns the bullet line, and `anyContainsWord` matches `D3` as a whole word anywhere in it. Result: correct, the prefix needs no parser change, as the summary claims.
- Stencil template filling: the edited `loom-template-discussion.md` lines add no `{{` markers, so `markers_test.go` marker sets are unaffected. Correct.
- Plan rubric: the edited line still has the "names the existing test" sentence after it, and the new rule keeps both existing finding triggers. Correct.
- Cross-text consistency: the template says "marked as the discussion spec's acceptance-criteria rule says", and the spec says `Review:`. The two agree and neither contradicts the rubrics.

**Findings:** none.

Tool calls: none; my context already held the diff, the validator code and the stencils.
