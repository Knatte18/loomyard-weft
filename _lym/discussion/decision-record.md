## Goal

```markdown
## Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 1.
In `test-only-functions-in-prod`, the Code-Review reviewer (6b616569, turns 18 to 35) wrote the branch's production diff of about 3,600 lines to `.scratch/prod.diff` itself, read it in five spans, and had its last Read refused for size, so it retried with `allow_large`.
The session cost $5.40 against a row median of $1.63, with the whole diff in its context.

## Idea

Burler writes the round's diff as one file per package, each under the Read cap, and names them in the explore step (`internal/burlerengine/prompt.go`, `contracts/stencils/burler/burler-step-1-explore.md`).
One session shows this; a second oversized review would confirm it.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - piece-cap-bytes-only (class design, severity MEDIUM, open in rounds 1)
  - split-order-and-assistant-pieces-unpinned (class design, severity MEDIUM, open in rounds 1)
  - staged-patch-not-committed-range (class design, severity MEDIUM, open in rounds 1)
  - stencil-overrides-rubric-range (class design, severity MEDIUM, open in rounds 1)
  - template-tests-lack-keep-markers (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
