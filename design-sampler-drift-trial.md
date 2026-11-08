# The sampler carries drift rows, off by default and switched on per task

`sampler-drift-trial` · Task · feature, drift, shed, loom · running · Code-Burler-Review

Imported from [issue #28](https://github.com/Knatte18/loomyard/issues/28).

## Seen

A `sampler` run cannot reach drift: every `lym drift` verb runs only inside a tapestry or darn run's worktree (`internal/driftcli/doc.go`), and the sampler recipe has no drift rows.
So drift's own stages (Drift-Baseline, Drift-Expect with its sealed expectation, the capture diff and the judge at Drift-Verify and Drift-Recheck, `lym drift check` and `lym drift amend`) can only be tried for the first time on a production task.

## Decision (operator, 2026-11-08)

The sampler recipe gets the drift rows tapestry and darn carry, switched off by default.
A sampler task can switch them on, so the operator can try the whole drift flow on a deliberate change before trusting it on real tasks.
With the rows off, a sampler runs as now.
The drift verbs run in a sampler worktree whose drift rows are on.
The stage gates of tapestry and darn stay as they are.

The switch is the existing `drift.scope` in the task's `run_settings`, with the sampler's default `off`, rather than a new key.
