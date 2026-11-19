## Goal

```markdown
Source: `docs/research/survey/pocock-skills-2026-11-19/report.md` (survey of Matt Pocock's skills repository, 2026-11-19), candidate 2, which holds the design.

## Idea

For an entry labelled `bug`, a gate step runs the tests the diff adds or changes (named by `testreach`) once on the merge-base tree and fails when every one passes there.
The fix commit carries a `Root-Cause:` trailer that Code-Review checks like `Loosened-Check:`.
A bug with no seam for a test is a stop the record names, never a silent pass.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-19): Red-first gate on darn runs only

Decision: Accepted: the red-first gate runs in Darn-Write only; tapestry and sampler runs of a bug entry are not checked in this run.

Rationale: Bug fixes run as darn by default on this hub; tapestry gates per Webster batch, a different seam with its own design questions, so extending the gate there is a separate change if bug runs on tapestry show the need.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - base-run-keeps-non-go-inputs (class design, severity MEDIUM, open in rounds 1)
  - entry-labels-has-no-behaviour-test (class design, severity MEDIUM, open in rounds 1)
  - gate-covers-only-darn-unrecorded-narrowing (class decision, severity MEDIUM, open in rounds 1)
  - judge-red-first-drops-named-failure-in-mixed-report (class design, severity MEDIUM, open in rounds 1)
  - merge-danger-blast-radius-too-narrow (class consistency, severity MEDIUM, open in rounds 1)
  - no-test-seam-and-bug-label-repeated-literals (class design, severity BLOCKING, open in rounds 1)
  - non-go-hub-red-first-exits-zero (class design, severity BLOCKING, open in rounds 1)
  - rubric-missing-trailer-clause-over-reaches (class consistency, severity MEDIUM, open in rounds 1)
  - verify-site-tripwire-and-docs-miss-new-site (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
