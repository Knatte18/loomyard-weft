---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The review proposed splitting over the failures just recorded, or over first with firstLog. I split over notes.flakyFailures() after addFlaky instead, which reuses the recorded flakyFailure values rather than rebuilding them from first and firstLog; a flaky pass ends the gate's evaluations, so the notes hold only this first run's failures (verifygate.go:491-499)."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer-report

## What changed

- F1: `TestWriteTriageFrictionNote` gains a "command failures only" subtest that compares the note by exact equality, with no flaky header.
- F2: `writeFailureOutput` opens and closes the block with a fence one backtick longer than the longest backtick run in the tail, never under three.
  The new helper is `longestBacktickRun`.
  A "a backtick fence in the tail" subtest asserts the four-backtick fence by exact equality.
- F3: the flaky-gate test's fake now returns `"rerun output only\n"` for the rerun, and the test asserts the friction note does not hold it.
- F4: the rerun-pass branch logs test and package ids under `flaky` and verify command failure logs under `verify_command_failed`, each only when non-empty.
  This differs from the proposal in the way the frontmatter's `reason` names.
- F5: "named no failing test" became "named no failing test or package" in `webster-spec.md`, `doc.go` and the doc of `triageWarnings`.
- F6: the landing describer stencil also says to name each verify command failure by its first run's log, as a failure of the verify step rather than a flaky test.
- F7: the pointer in `loom-final-summary-spec.md` now says the section covers identities, and verify command failures, that passed on rerun.

The gate then flagged three comment lines the round added: two over-long lines at `doc.go:993` and `triage.go:41`, and a comment in `writeFailureOutput` that broke mid-sentence.
A first repair split the two long lines at a comma, which the gate read as a mid-sentence break.
The final repair rewrites each of the two as two short sentences, and the third comment as one sentence per line.

`lym gate test ./internal/websterengine` and `lym gate test ./contracts/stencils` pass, and each fix has its own commit.
The F2 test row sits in the commit of F1, since both rows live in `triage_test.go`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings; its note that the stencil package's own tests were unrun is settled, since `lym gate test ./contracts/stencils` passes.
- fix-affected-set: LOW, the notes type's doc wording says "failures", so it is right and needs no change (no change).
- fix-affected-set: LOW, `summary.go:48-57` already lists command failures apart, so it needs no change (no change).
- fix-affected-set: LOW, the rerun-pass branch reads every recorded failure, which is latent only because a pass ends the evaluations (no change, as the review's Rejected section holds).

## Disputed

None.

## Deferred

None.

## Rulings

None.
