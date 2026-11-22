## Goal

```markdown
## Seen

`darn-slowdown-measure` (`docs/research/darn-slowdown-measure.md`) found that each verify gate spends about 75 to 105 s after its tests:
the reach, redundancy and mutation reports and the red-first step run one after another once the tiers finish.
Over a darn's verify gates that is 4 to 7 % of a 45 min run.

## Goal

Run the three reports concurrently with the tier 2 tests, or only on the last verify gate before a handoff, whichever the code supports cleanly; keep red-first where it is.

## Bounds

Time one verify gate of the same diff before and after the change, once each, and state both wall times in the change description.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-22): No before-and-after verify timing

Decision: The entry's bound to time one verify gate before and after the change is dropped; no timing is made in this run.

Rationale: An after-timing needs a lym built from this branch, which neither the run nor the orch runs; the gain is measured from archived run records after landing.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - gateslot-doc-sequential-tail (class consistency, severity MEDIUM, open in rounds 1)
  - guard-foreign-wiring-unpinned (class design, severity MEDIUM, open in rounds 1)
  - last-tier-start-untested (class design, severity MEDIUM, open in rounds 1)
  - pattern-test-speed-background-sequential-tail (class consistency, severity BLOCKING, open in rounds 1)
  - report-steps-share-last-tier-process-cap (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
