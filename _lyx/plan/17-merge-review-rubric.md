# Card 17 — merge-review-rubric

**Create:**
- `contracts/stencils/loom/loom-rubric-merge-review.md`
- `contracts/stencils#LoomRubricMergeReview`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils#TestLoomRubrics`

**Intent:**
Add the rubric stencil the Merge-Review perch runs (D2), ahead of the recipe rows that reference it.

The rubric stencil `loom-rubric-merge-review` asks the reviewer to judge only the interaction (D2).
- Invariants the parent's commits added or changed are judged against the task's code, and the task's code against the parent's new code.
- The subject is the `{{.merge_review}}` note: the two ranges, the merge commit and the overlap, or the fallback naming the latest merge-in.
- The fasit is `PATTERN.md`, the overlap's `pattern/` files and the decision record.
- Do-not-flag excludes anything the Webster review already judged on the task's own range.
It is registered in `entries`, `rubricMarkerAllowlist` admits `merge_review`, and `TestLoomRubrics` gains its row.

**ImpactSummary:** a new rubric stencil no recipe row runs yet; the stencil registry, the marker allowlist and the rubric test each gain one entry.

**Commit:** 17: stencils: the loom-rubric-merge-review rubric
