## Goal

```markdown
## Seen

Report `docs/research/turns/batching-hook-remeasure-2026-11-05` (d05c8d75): the hook fired 13 times in 22 runs; 156 edit follows met 2 edit firings, while a replay of the hook on one recorded edit follow fires. Edit follows rose from 4.2% to 6.9% of context (Webster forks 6.9% to 11.2%), all mid-round, none after a failed gate. The report recommends removing the hook and names the live gap as its one open question.

## Idea

Find why the hook does not fire live on edit follows before deciding to remove it: either fix the live gap and measure again, or remove the hook. Mid-round edit splitting is where the edit-round rule fails, so a fix there is what matters.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - constructors-transcript-wait-unpinned (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
