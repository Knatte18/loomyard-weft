## Goal

```markdown
## Seen (orch, 2026-11-17)

Runs last hours to days and stencils change often, so a resumed run's later producers may get prompts that differ from those it started with,
and nothing says so.
Source: the survey F17 (`orchestrator-survey-ideas`), from Conductor's `bundle build` and Bernstein, which refuses to resume after its spec digest changes.

## Idea

- A run records the digest of each stencil it may render when it starts, in weft.
- Resume compares them with the installed stencils and warns the orch of each that changed, naming `lym stencil diff`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - notice-overclaims-stencil-diff (class consistency, severity MEDIUM, open in rounds 1)
  - pins-left-uncommitted-after-failed-seed-commit (class design, severity MEDIUM, open in rounds 1)
  - seed-commit-of-pins-untested (class design, severity MEDIUM, open in rounds 1)
  - start-stencil-check-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
