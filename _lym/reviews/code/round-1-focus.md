---
round: 1
focus:
  - "contracts/stencils/loom/loom-template-discussion.md: does the bug paragraph tell the writer to read the entry's labels from what Step 1 already reads, and is `bug` detectable there? Does it leave every other rule of the stencil intact?"
  - "contracts/specs/loom-discussion-spec.md `### A bug entry's record`: does it match the red-first gate's real behaviour (`internal/gateslot/redfirst.go`, `internal/loomshed/redfirstgate_test.go`) on the `No test seam` waiver and on the merge-base test, and does it sit in a section consistent with the spec's other rules?"
  - "Rubric bullet in contracts/stencils/loom/loom-rubric-discussion-review.md: does it agree with the spec's three requirements (Goal, Constraints, Acceptance criteria), neither adding nor dropping one?"
  - "Do the spec, stencil and rubric together keep a bug record from waiving the gate itself, and do they avoid restating the spec's format in the stencil?"
  - "Commit message and summary.md claims against the diff; the commit message carries no `Root-Cause:` trailer, so check whether the answer key asks for a bug fix at all, as against a process change."
---

Prose-only diff: three contract files, no Go change, so the reach and mutation reports are empty.
