## Goal

```markdown
## Seen (2026-11-26, `config-keys-nested` at Webster)

Webster's Master wrote its friction note at `_lym/loom/friction/webster-master.md`, the path the friction-directive-orchestrator stencil gives it.
The run-exit audit (`internal/websterengine/audit.go`, class `parent-write`) allows Master only `outcome.yaml` and `summary.md`, so it flagged the note and blocked the run with all 8 batches done.
`lym webster accept-audit` refused twice:
first because HEAD carried the verify fixer's commit past the last batch head, and after `reset --to last-batch-head` and removing the note,
because the note's path "cannot be checked: outside the task worktree", since the records side is a separate worktree.
Its only way forward was `lym webster reset --to start`, which throws away every batch.

## Idea

- The parent-write audit exempts Master's own friction note path, the one its prompt names.
- `accept-audit` accepts a finding whose path lies on the run's records side and is absent or unchanged, instead of refusing it as outside the worktree.
- A verify fixer's commit past the last batch head does not block `accept-audit`, as a parent merge does not.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - fixed-head-tolerates-unvetted-commits (class design, severity MEDIUM, open in rounds 1)
  - master-note-path-save-and-read-untested (class design, severity MEDIUM, open in rounds 1)
  - master-stencil-still-limits-to-two-contract-files (class consistency, severity MEDIUM, open in rounds 1)
  - restarted-gate-loses-prefix-head (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
