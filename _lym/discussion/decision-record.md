## Goal

```markdown
## Seen (raddle of `sandbox-closed-area-build`, 2026-11-26)

The stored record's D6 attach criterion runs over two lines, the second not indented under the bullet.
The record parser keeps only the first line as the criterion.
Raddle could not carry the whole bullet: joined to one line, the raddle-propose gate refused it as criterion-unknown, since it matches only the cut text.
The second clause survived only by being restated in the part's contract.

## Idea

Either the record format check refuses an unindented continuation line when the record is stored,
or the parser reads it as part of the bullet, as markdown does.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - continuesitem-excludes-too-many-hash-and-angle-lines (class design, severity MEDIUM, open in rounds 1)
  - html-and-blockquote-exclusions-unpinned (class design, severity MEDIUM, open in rounds 1)
  - merge-danger-blast-radius-too-narrow (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
