# Webster run-exit audit flags Master friction note as a parent write, and accept-audit cannot clear it

`audit-flags-master-friction` · Task · bug, webster · starting · Worktree-Create

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
