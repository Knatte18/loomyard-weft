## Goal

```markdown
## Seen (orch, 2026-11-01)

In the five heaviest Webster forks of the three latest tapestry runs, about 10 turns went only to reading gate output: the result was too large for one tool result and was saved to a file, or a `tail` showed only passing tests.
Lint ran in turns of its own after the gate (`lym loom lint-comments`, `lint-pattern`): 14 lint-only turns, and fixed-column comment findings cost about 8 more.
A fork fixed one failure, ran the whole gate again and read the next, rather than fixing every failure at once.

## Idea (operator, 2026-11-01)

The gate's report is written so that every failure can be fixed in one go:
- `lym gate test` prints one compact list of every failure: build errors, test failures and lint findings, the gate's own lint steps included.
- Each failure names its package, test and `file:line`, the expected and actual values or the first lines of the panic or build error.
- It names the files the change touches and the path of the full log, and stays small enough to read as one tool result.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - build-failure-report-untested (class design, severity MEDIUM, open in rounds 1)
  - lint-exit-code-and-halt-report-untested (class design, severity MEDIUM, open in rounds 1)
  - lint-findings-cut-first-cut-line-wrong (class design, severity MEDIUM, open in rounds 1)
  - touched-files-hub-branch-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
