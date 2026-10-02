# Card 13 — webster-review-rework-range

**Prosa:**
- `contracts/stencils/loom/loom-rubric-webster-review.md`

**Edit:**
- `contracts/stencils/rubric_test.go`

**Create:**
- `internal/loomcli/reviewrange_integration_test.go`

**Intent:**
Narrow Webster-Review to the live generation's own commits in a rework generation (D10), since the live plan is the answer key and earlier generations' code already passed review.

The rubric's "Determining the review range" section gains a rework branch, still the single definition of the range:

1. Find the live generation's round as Plan-Review does: among `_lyx/loom/rework/round-<N>/` directories, the highest `N` whose `record.json` `first_card` equals the `first_card` in `_lyx/plan/00-overview.md`'s frontmatter (absent means `1`).
2. With such a round, take its `record.json` `head_sha` (the rejected PR head) and review the branch's own commits after it, excluding the parent branch's changes a mid-run merge brought in: `git log --first-parent --no-merges -p <head_sha>..HEAD`.
3. With no such round, the range stays `git diff $(git merge-base <product.parent> HEAD)..HEAD`.

Earlier generations' code is context, never the subject.
An unreadable `record.json` or an empty `head_sha` in the live round raises the same BLOCKING "could not be determined" finding as an unreadable status file.

`reviewrange_integration_test.go` (tag `integration`, this package's hermetic git `TestMain`) settles the git form against a real repository: base commit on the parent; a branch commit `A` (the rejected head); a parent commit `P`; a branch merge of the parent `M`; a branch commit `B`.
It extracts the rework-branch command from `stencils.LoomRubricWebsterReview` (the line beginning `git log --first-parent`), substitutes `<head_sha>` with `A`, runs it, and asserts it lists `B` alone, never `P` or `M`.

Tests in `rubric_test.go`: the webster rubric names the rework branch's `first_card` round rule, `head_sha`, the `--first-parent --no-merges` form, and keeps the `git merge-base` fallback.

**ImpactSummary:** stencil-test assertions plus one new integration test; the rubric text changes what the Webster-Review agents read, and no Go code reads it.
