# Record the modules a landed task touched, for reports

`labels-from-diff` · Note · enhancement, board, loom

## Seen (count, 2026-10-15)

Of 50 landings on `main` from 2026-10-05 to 2026-10-15 that changed non-test Go code in `internal/shuttleengine/*.go`, 47 were matched to their task through the closing PR comment, and only 24 of those carried the `shuttle` label.
The others carried labels for what the task was about (`reed`, `loom`, `burler`, `batten`, `stencil`), not for every package its diff touched: `run-pane-timing`, `overview-seats`, `hibernate-trigger`, `merge-review`, `twill-discusser` and `small-fixes-batch` among them.
The board's module labels are set by hand when an entry is written, before anyone knows which packages the work will touch.
The orch bundles tasks and sets `depends_on` by package overlap read from labels, so a label that misses a touched module hides an overlap.
Counted by Haiku agents from the board history and `gh pr list`; scripts and tables in `.scratch/label-count/` in the prime, not committed.

## Idea (operator, 2026-10-29)

A label is useful only before the run starts, when the orch reads it to bundle tasks and set `depends_on`.
Deriving a task's labels from the plan's `Edit:`/`Create:` paths or from the landed diff comes too late for that, so it is dropped.
What remains is a fact for reports: record the modules a landed task touched on its entry or in its run record, so a report reads what the diff touched and not the labels written up front.
The labels themselves stay the work of `label-coverage`: a label for every module, so the right one exists when the entry is written.

## Open

- Whether the touched-modules record is worth building at all once every module has a label.
