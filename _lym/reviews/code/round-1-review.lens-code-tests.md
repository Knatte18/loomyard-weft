```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff changes, I looked for the test that pins it.

**Per behaviour:**
- **The clause removed from `contracts/stencils/engine/claude/edit-directive.md:9`.** No test. This is a stencil's static prose, so it is exempt. `internal/shuttleparity/compare_test.go:110-119` renders the directive and checks only the manifest item. It never asserts on the wording. A grep of `internal/editdirective/*.go` found no "harness" or "cat" assertion in `editdirective_test.go`.
- **The sentence changed in the package comment at `internal/editdirective/doc.go:13`.** No test. It is a Go doc comment with no runtime behaviour, so it is exempt as static prose.
- **The new `An unpinned tool fact` bullet at `contracts/stencils/loom/loom-rubric-discussion-review.md:43-46`.** No test. It is rubric stencil prose, so it is exempt.
- **New or changed tests in the diff.** None. The diff stat lists only the three files above, so there is no redundancy or prose-assertion candidate.

**Findings:** none.

**Tool calls:** none. My inherited context already held the diff, the directive tests and the parity test.
