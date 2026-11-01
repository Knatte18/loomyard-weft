## Goal

```markdown
## Seen (orch, 2026-11-01)

`tools/tokencount` with no slugs counts only finished full tapestry runs, those that reached Finalize under the tapestry recipe.
On 2026-11-01 it skipped the 80 most recent runs, most of them darns, and counted three tapestry runs from 2026-10-31 and 2026-11-01.
The operator prefers parallel darns to one long loom as cheaper and faster, but no report gives a darn's cost beside a tapestry's.

## Idea

tokencount counts darn runs too, with darn's own segments (writer, Code-Review rounds, landing), and can compare recipes, for example the cost per run and per changed line of darns against tapestries over the same period.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - since-path-untested (class scope, severity MEDIUM, open in rounds 1)
  - unparsable-description-aborts-report (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
