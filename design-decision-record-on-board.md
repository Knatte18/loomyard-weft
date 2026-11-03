# An approved Discussion stores its decision record on the board entry and flags it discussed

`decision-record-on-board` · Task · feature, board, loom

## Seen

A task's decision record lives only in its run's weft (`_lym/discussion/decision-record.md`): a cancelled run leaves it in the records history, and no new task takes it along.
This task is the first half of the core of note `discussed-task-split`: storing the record and flagging the entry.

## Wanted

- When Discussion is approved, after its review, lym copies the decision record onto the task's board entry and flags the entry discussed.
- `lym loom decision add` updates the stored record the same way.
- lym itself writes the record and the flag, so both are exempt from the lock on a claimed entry: that lock exists so the orch does not change a task's text after its agents have read it, and a record written by lym is not that.
- `lym board get` shows the flag and can print the record; a cancelled run leaves the record and the flag on the entry.
- Where the record lives is for Discussion to settle: in `board.json`, or in a file of its own in the Bolt checkout beside it with the same entry-level replay, since a record is often 10–30 KB.

## Out

- Skipping Discussion for a discussed entry: that is a gate check on the `cartoon` block of task `shed-sub-run-block`, a follow-up task that waits on both.
- Tapestry and darn are unchanged: a discussed entry still runs its Discussion until that follow-up lands.
- Splitting a discussed entry into tasks.
