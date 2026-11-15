```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour, the test that pins it:**
- **`raddle-template-agent.md`, Step 3's three new bullets (rule 1, rule 2, rule 3):** None. This is static stencil prose, so it is exempt.
- **`loom-rubric-raddle-review.md`, "The cut runs the most parts at once" bullet:** None. Static prose, exempt.
- **`loom-rubric-raddle-review.md`, "No part guesses at another's interface" bullet:** None. Static prose, exempt.
- **`loom-rubric-raddle-review.md`, the rewritten `design` class sentence:** None. Static prose, exempt.
- **Markers in both stencils:** The diff adds no `{{.X}}` marker and no `{{if}}` or `{{range}}` to either file, so the marker set is unchanged. `contracts/stencils/rubric_test.go:53`, the `loom-rubric-raddle-review` row, still pins the marker allowlist and the writer-stencil name for the rubric, and nothing in the diff changes either one.

**Findings:** None. The diff changes only static prose in two stencils. That is exempt from the uncovered-behaviour bullet, and it adds no test, so it has no redundancy candidate and no new assertion on prose. The redundancy report lists no new or changed Go test for this diff.

**Tool calls:** None.
