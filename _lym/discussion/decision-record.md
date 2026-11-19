## Goal

```markdown
## What was seen (2026-11-18)

`comment-trim-second-pass` (darn, profile `docs`) measured comment shares and wrote the result only into its change description (`_lym/landing/summary.md`).
It committed nothing to the repository, so Land landed an empty diff and dobby reported "landed as 64e58795", which was main's tip.
The finding then lived only in the archived records tag (`archive/comment-trim-second-pass/...`), where no one looks.

## Idea

- The darn writer stencil for profile `docs` requires the report as a committed file under `docs/research/`.
- Land refuses, or at least flags in dobby's line, a run whose code diff against its parent is empty.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - empty-diff-test-lacks-parallel (class consistency, severity BLOCKING, open in rounds 1)
  - empty-landing-parent-records-commit-counts-as-changed (class design, severity BLOCKING, open in rounds 1)
  - restep-after-failed-push-overwrites-empty-flag (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
