## Goal

```markdown
Imported from [issue #164](https://github.com/Knatte18/loomyard/issues/164).

## What happened

A reviewer-fix agent found that `namesRoot` (`internal/hardenshed/reap.go`) does not match an orphan process whose working directory sits under a worktree's `_lym` link.
`/proc/<pid>/cwd` reports the link's target, so the comparison against the root path never matches.

## Why it is abnormal

A sandbox process started with such a working directory and no root on its command line escapes the reap.
The orphan-kill test failed for this reason until it was changed to name the root on the command line, which hides the gap instead of closing it.

## What to do

Resolve symlinks on the root (or compare against both the link path and its target) in `namesRoot`, then add a test whose orphan has the `_lym`-link working directory and no root on its command line.

Provenance: task `hardener-campaign`, from `burler-code-r1.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - reap-root-resolution-outcomes-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
