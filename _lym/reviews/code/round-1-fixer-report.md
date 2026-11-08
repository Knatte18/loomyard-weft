# Fixer-report, round 1 (code)

Every finding of `round-1-review.md` has a disposition.
The gate `lym gate test ./internal/loomcli ./internal/landingshed` passes after the last change.

## Changes

- **F1 (MEDIUM), fixed differently.**
  The proposed change is a `ParentReviewAnswered` flag in the record, with `judge` holding while the ask has no satisfied answer.
  I did not make it: the orch answers by `lym creel post` to the PR-Review session's strand, so the answer is creel mail to the session's agent.
  The row never reads that mail, and no seam or verb tells it the orch is satisfied, so a flag has nothing to be written from.
  A gate would need a new channel from the agent to the record, which is a design the review does not specify.
  I took the review's alternative reading instead: the row asks and does not gate.
  `internal/landingshed/doc.go` and `docs/overview.md` now say so, in place of the line "The row's endings do not wait on the orch's answer", and the `askParentReview` doc comment says it too.
  I recorded no `lym loom decision add`, which is the parent's or operator's to run.
  The board entry's wording that the run waits at PR-Review until the orch answers is therefore not met by the code, and I flag it here: the parent should settle whether a gate is wanted.
- **F2 (LOW), fixed as proposed in code, test row deferred.**
  `askParentReview` logs a failed `updateRecord` with `logger.Warn` and no longer returns an error, so a record write that fails after a successful post does not make the driver re-step and post again.
  Since every path then returned nil, the function's `error` return and the caller's error branch were dead, and I removed both after the self-check.
  The doc comments in `prreview.go` and `doc.go` list the failed write among the logged cases.
- **F3 (LOW), fixed as proposed.**
  `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers` in `internal/loomcli/landingdeps_test.go` builds `pullRequestDepsGuardedBy` over a fixture hub.
  It asserts `ParentReviewWanted` reads false with no layer setting it, true with the worktree's `loom.yaml` set, and true with the task's local layer set over a `loom.yaml` of false.

Commits, with F1 and F2 sharing one because both touch `prreview.go` and `doc.go`:

- `landingshed: fix F1, F2 — PR-Review asks and does not gate on the orch's answer; a failed ask-record write is logged, not re-stepped`
- `loomcli: fix F3 — pin that ParentReviewWanted reads pr_review_ask_parent over the pair's layers`
- `landingshed: fix F2 — drop askParentReview's dead error return; loomcli: name the layers the F3 test rows set`
- A gate finding flagged the 120-character doc line above `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers`.
  A second gate finding then flagged my first split as a mid-sentence wrap.
  The doc comment is now one sentence per line, none near the limit, in a fifth commit.

## Self-check

- fix-code-correctness, LOW: `askParentReview`'s error return and the caller's branch are dead. Fixed as proposed.
- fix-code-correctness, LOW: the F3 row named "the hub layer" writes the worktree's `loom.yaml`. Fixed as proposed; the row names now say what each sets.
- fix-code-correctness, LOW: F2 has no regression row. Deferred, see below. Merged with the fix-proposal finding of the same content.
- fix-proposal, LOW: the F2 test row is missing. Deferred, see below.
- fix-proposal, LOW: the "hub layer" row name overstates what is pinned. Duplicate of the correctness finding; fixed with it.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- **F2's test row** for a record write that fails after a successful post.
  The fixture has no seam that fails the record write alone.
  Removing the record inside the ask seam also fails the wait's own record read, and replacing the lock path fails every read, so such a row passes with the old defect in place and pins nothing.
  A row needs a record-write seam in `PullRequestDeps`, which is a production change the finding does not ask for.

## Rulings

None reached me this round.
