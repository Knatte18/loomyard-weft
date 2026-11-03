## Goal

```markdown
## Seen (2026-11-03)

Task `codex-engine` ran recipe `motif` to write its Sketch, then the orch turned the entry into a project (recipe `pawl`) to split it into parts.
`lym beam create codex-engine` refused: `"codex-engine" has already completed`, naming `lym batten cleanup --apply` and deleting the old task branch.
The workaround was `lym batten cleanup --apply` from the prime, after which `lym beam create codex-engine` started.

## Expected

A motif run is the planned first step of a project split, so its done batten record does not block the project's beam.
`lym beam create` clears the slug's done record itself when that record's run was a motif run whose pair was torn down, or the motif run removes its record when it lands.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-03): No backfill for motif runs done before this change

Decision: A done motif run seeded before this change, with no child-recipe record, keeps the old refusal; no legacy path reads the seed or board type to clear it.

Rationale: The one such run, codex-engine, was already cleared by hand with lym batten cleanup --apply and its beam is running; no other done motif record exists, and lym batten cleanup --apply stays the way forward for any stray one.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - clear-motif-run-commit-failure-half-removal (class design, severity MEDIUM, open in rounds 1)
  - seed-child-child-recipe-write-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
