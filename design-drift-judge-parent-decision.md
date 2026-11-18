# drift judge halt blames the judge for a parent's lym loom decision add during Drift-Verify

`drift-judge-parent-decision` · Note · bug, drift, loom

Imported from [issue #145](https://github.com/Knatte18/loomyard/issues/145).

## Seen

While a darn run's Drift-Verify judge session was running, the parent recorded a decision with `lym loom decision add <slug> --by parent`.
The verb committed `_lym/discussion/decision-record.md` on the records side.
When the judge finished, the run halted with `decision halt: the judge session changed what it may not: records: ... decision-record.md was changed`.
The judge had not touched the file; the parent's own verb had.

## Workaround

The parent restored the record to the snapshot HEAD in the records worktree (`git checkout <snapshot head> -- .../decision-record.md`), deleted the capture's scratch directory, ran `lym loom resume`, and records the decision again after Drift-Verify ends.

## Expected

`lym loom decision add` is either refused while a drift judge session runs, naming when to retry, or its commits are excluded from the judge's snapshot check.
