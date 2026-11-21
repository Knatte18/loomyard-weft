## Goal

```markdown
Imported from [issue #159](https://github.com/Knatte18/loomyard/issues/159).

A burler fix agent reported two frictions in a code round that applied a review's findings.

1. The per-finding commit rule fought the change set.
   Several findings touched the same files as each other and as other findings in the round.
   The agent grouped findings by file and named each group in the commit, which departs from the per-finding rule.
   The burler fix instructions give no sanctioned way to commit findings that share files.

2. A finding's `location` key did not name the seam the fix had to reach.
   The agent had to read several unrelated files to find the real site before one edit could start, which cost most of the round.
   Separately, the fix-check rubric and fork results surfaced two problems that the review's `fix` keys did not show:
   a timeout setting mismatch on the judge call, and missing enforcement of parent findings.

Why it is abnormal: the fix instructions should let a round commit shared-file findings without improvising,
and review findings should point at the code site the fix needs, or say that the site must be found.

What to do:
- Decide the commit rule for findings that share files, and state it in the burler fix prompt.
- Consider having the review's `location` and `fix` keys carry the seam a fix reaches, or let the review flag findings whose site is not named.

Provenance: task `parent-review-code-and-drift`, note `burler-code-r1.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - joint-commit-groups-do-not-chain (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
