## Goal

```markdown
From F17's Kiro lead (bugfix spec with a fail-on-base test).
The Go gate half has landed: `lym gate red-first` on the `Darn-Write` row, and the `Root-Cause:` trailer.

## Idea

When Discussion-Write runs for an entry labelled `bug`, its stencil asks for and records:

- the observed fault and how to reproduce it;
- the root cause, or what is still unknown about it;
- the test that will fail on the merge base and pass after the fix, or why no test seam exists.

The decision record then states up front what the red-first gate later checks.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - bug-record-rule-unscoped (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
