## Goal

```markdown
## Seen (operator, 2026-11-03)

The operator often works on another machine while this one sleeps, whether the sleep came from `lym hibernate request` or from the machine itself.
When origin or a run's owner moved meanwhile, a run that goes on after the wake works on a stale base, and nothing tells it until a push is refused or a landing conflicts.

## Idea

The check belongs where the stale base does harm, not at the wake (operator, 2026-11-03):

- Before a run commits for a push, pushes, or lands, lym fetches and compares the run's base and parent branch, and the owner of the run, its beam and root, with what the run last saw.
- When an owner moved to another machine, the run stops before it pushes, and the stop names `lym adopt`.
- When only origin's tips moved, the run takes the new base the way a landing already does, or stops naming what moved when it cannot.
- The orch hears what moved in one short notice line.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - code-repo-literal-duplicated (class consistency, severity BLOCKING, open in rounds 2)
  - parent-notice-path-outside-shedrun (class consistency, severity BLOCKING, open in rounds 2)
  - pr-review-pushtask-pausepair-untested (class scope, severity MEDIUM, open in rounds 1, 2)
  - skip-closure-untested (class scope, severity MEDIUM, open in rounds 1, 2)
  - stopped-check-callers-unpinned (class scope, severity MEDIUM, open in rounds 1, 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
