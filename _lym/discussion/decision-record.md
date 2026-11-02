## Goal

```markdown
## Seen (operator, 2026-11-02)

Operator-facing verbs such as `lym adopt root` print one long JSON envelope: on a stop it carries every commit hash of a divergence and every run's record, so the operator cannot see what happened or what to do next.

## Idea

- A verb the operator types prints a short human summary by default: what it did, what stopped, and the way forward for each stop, one line each.
- The full JSON envelope stays available behind a flag, for agents and scripts.
- Long lists, such as a divergence's commit hashes, collapse to a count with a pointer to where the full list is.
- Which verbs count as operator-facing follows the command index's operator audience.

## Open

- Whether agents call these verbs too, and so need the flag added to their stencils and skills.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - note-key-list-collapses-to-count (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
