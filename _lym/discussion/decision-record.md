## Goal

```markdown
## Seen

The landed `push-retry-not-halt` (`3fb0787e3`) records that a dobby catch-up whose fetch fails on an unreachable origin still stops that run's start,
and leaves to the parent whether that stays. Overnight runs then wait for an orch to run `lym batten run` again.

## Idea

Treat a catch-up fetch on an unreachable origin as a transient network stop: dobby retries the start on its own backoff and reports only past the bound,
as the pushes now do.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - beam-catchup-wait-missing-from-push-summary (class design, severity MEDIUM, open in rounds 1)
  - non-origin-catchup-failures-lose-start-refusal (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
