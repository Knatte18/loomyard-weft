## Goal

```markdown
Imported from [issue #286](https://github.com/Knatte18/loomyard/issues/286).

On `lym board sync` from the prime, runpusher logs one WARN per task worktree that was removed long ago:

`runpusher: the pusher did not start ... resolve main worktree: list git worktrees in "<hub>/<slug>": dotgit: not a git repository: no .git above <hub>/<slug>`

Five such slugs were listed, all of runs that landed and whose pairs were removed through lym fabric. The pusher's run registry keeps entries for worktrees that no longer exist. Expected: entries for removed pairs are dropped at removal (or skipped silently when the directory is gone).
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - merge-danger-understates-prune-deletion-and-callers (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
