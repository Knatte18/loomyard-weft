## Goal

```markdown
## Seen (adopt-build beam orch, 2026-11-16)

`lym beam pr open` opened beam `adopt-build`'s pull request #132 ready for review while it was "dirty", conflicting with `main`, and while a merge-in member still had to land.
The operator could have squash-merged a pull request whose work was still landing.

## Decision (operator, 2026-11-16)

A beam's pull request must not be opened before its conflicts with the parent are fixed.

## Idea

`lym beam pr open` refuses, or runs the merge-in first, while the beam conflicts with its parent or a member is still landing;
or it opens the pull request as a GitHub draft and marks it ready only at `lym beam ready` or once it is mergeable.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - pr-open-help-omits-new-refusals (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
