# Solo fix review for an over-cap Finalize fixer change

A merge fixer change at Finalize that is over the caps (`FixCap.Exceeded`) used to reroute the run to Parent-Align and the full Code review segment, however small the change.
A ten-line test lock in `creel-hook-post` cost a whole `solid` lens fan.
Now, when a review follows Finalize's reroute, Finalize first spawns one solo fix review session that reads only the fixer's commits, `Baseline..HEAD`.
An approval lets Finalize go on and land; the caps themselves are unchanged.

## Routes

- `approved`: the run lands with no reroute, and the approved HEAD is kept in the attempt record's new `reviewed` field, so a later pass of the entry at that HEAD lands without a second review.
  A HEAD that moved since the approval is reviewed again.
- `rejected`, or a session that ends with no usable verdict (timeout, no report, an unparseable report after one re-prompt): reroutes along Finalize's `on_reroute` edge as before, the reason adding the cap, the fixer's commits and the reviewer's findings or why there is no verdict.
- A session that leaves the tree dirty or moves the task head: halts with `LandingStopPrefix`, since the reviewer may not change the branch.
- A session that fails to start: a transient error, as the fixer's is.
- Unchanged, with no fix review: a spent fixer still reroutes, and an over-cap change with no review after the reroute (Code budget 0, a pawl run) still halts for the orch.

## Design decisions

- The review runs inside Finalize's row, under the parent lock, the way the merge fixer does.
  A recipe row would need a new edge in every recipe and would release and retake the lock around a short read-only session.
  The lock is held for at most one more `fixer_timeout_min` session.
- The session uses landing.yaml's `fixer` model-spec and `fixer_timeout_min`; no new config key.
- The prompt, a new stencil `landing-template-fix-review.md`, asks whether the change fixes the failure without weakening, skipping or deleting a test, loosening an assertion or bending gate or build configuration, and whether it is otherwise correct.
  It points at the new report spec by name.
- The report format is pinned by `contracts/specs/landing-fix-review-spec.md`: a first line `verdict: approved` or `verdict: rejected`, then a `## Findings` section of bullets, non-empty on a rejection.
  `ParseFixReviewReport` in `internal/landingshed` is its sole parser, recorded in `pattern/PATTERN-sole-parsers.md`.
- Every uncertain end falls back to the full review, so nothing lands on less than a reviewer's explicit approval.
- Risk: a reviewer that approves a change the full Code review would have rejected lands it.
  The bound is the prompt, which names what the caps guard.

Docs follow: `internal/landingshed/doc.go`, the Finalize comments in `loom-recipe.yaml` and `darn-recipe.yaml`, `docs/overview.md`, and the `fixer` key descriptions in `llm-model-spec.md`, `config.go` and `template.yaml`, which now cover the fix review session too.

## To check by hand

No live run of a fix review session against a real model was made; the tests drive Finalize through the fake seat runner.
Check that a real session writes a report that `ParseFixReviewReport` accepts.
Also check:

- `finalize_gate_test.go`: the approval writes `Reviewed` to the attempt record, and a HEAD that moved since the approval is reviewed again.
- `fixreview.go`: a review timeout, a failed session start and the single re-prompt over an unparseable report route as described above.
- `finalize_gate_test.go`: a spent fixer spawns no fix review session.
- `fixreview.go`: the session runs under the `fixer` model-spec and a `fixer_timeout_min` turn bound.
- `finalize.go`: the halt for a reviewer that dirtied the tree names only the way forward of restoring the fixer's range, with no contradictory "commit then resume" tail.
- `llm-model-spec.md` and `docs/overview.md`: the fixer keys are described as governing the fix review session too.
