## Goal

```markdown
## Seen

2026-11-19: raddle's proposal for `single-repo-records-branches` passed its checks and the operator approved it, but Raddle-File refused the part `single-repo-records-on-code-removal` because its slug is 35 characters and the board's limit is 32.
The run stopped with a question to the parent.

## Idea

The proposal checks apply the board's slug rules, the length limit included, so a slug too long is caught before the operator approves.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-19): No test seam for the red-first overlay: the diff's spec change blocks it

Decision: The red-first gate is waived for this run. The bug has a Go seam (TestRaddleBoardContext and the new TestCheck rows fail on the merge base), but the diff also changes contracts/specs/raddle-proposal-spec.md, a required doc update the base overlay cannot put back.

Rationale: The block lies in the red-first overlay's handling of non-Go files, not in a missing seam; the overlay limit is filed as its own board note.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - junction-names-wiring-unpinned (class design, severity MEDIUM, open in rounds 1)
  - kept-detail-test-lacks-keep-marker (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
