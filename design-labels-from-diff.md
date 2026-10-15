# Module labels from what a task touches, not only what it is about

`labels-from-diff` · Note · enhancement, board, loom

## Seen (count, 2026-10-15)

Of 50 landings on `main` from 2026-10-05 to 2026-10-15 that changed non-test Go code in `internal/shuttleengine/*.go`, 47 were matched to their task through the closing PR comment, and only 24 of those carried the `shuttle` label.
The others carried labels for what the task was about (`reed`, `loom`, `burler`, `batten`, `stencil`), not for every package its diff touched: `run-pane-timing`, `overview-seats`, `hibernate-trigger`, `merge-review`, `twill-discusser` and `small-fixes-batch` among them.
The board's module labels are set by hand when an entry is written, before anyone knows which packages the work will touch.
The orch bundles tasks and sets `depends_on` by package overlap read from labels, so a label that misses a touched module hides an overlap.
Counted by Haiku agents from the board history and `gh pr list`; scripts and tables in `.scratch/label-count/` in the prime, not committed.

## Idea

- Derive a task's module labels, or check them, from what the work touches: the plan's `Edit:`/`Create:` paths once Plan-Write has run, and the landed diff at landing.
- At least, record the touched modules on the landed entry or in the run record, so overlap checks and reports read facts, not the labels written up front.
- Whether a mismatch only warns, adds the label, or feeds the orch's overlap check is open.
