# driver-context brought the loom-status pane back on llm-driven runs

`retire-loom-status-strand` · Note · bug, loom, reed

## Seen

`driver-context` (f5b5218f7) brought the `loom-status` strand back on llm-driven runs: `lyx loom start` now ensures the status strand on every run, where it used to remove it on an llm-driven one (`internal/loomcli/start.go`, its step 2 and `mustUseLLMDriverArm`).
Runs started on the new binary, `quick-recipe` and the restarted `cli-surface` (2026-10-10), again show a `loom-status` pane at the top of their window; runs started before it do not.
The operator reads a run's state from its driver pane, and llm-driven runs had shown no status pane for a long time.

## Idea

Restore the earlier behaviour: an llm-driven run removes the status strand, and only a go-driven run keeps it.
