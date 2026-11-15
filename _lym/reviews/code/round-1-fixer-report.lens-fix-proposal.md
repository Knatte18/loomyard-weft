No findings.

I tried to build a case where the done-entry row (F1) stays silent for a reason other than the `isDone(after)` guard.
- `Board.SetStatus` (`board.go:212`) only sets the status field through `store.SetStatus`.
- The only place that clears `Discussed` is `render.go:72`, which touches the rendered copy and not the store.
- Nothing in non-test code clears `RecordBodySHA256`.

So a done, discussed, stored-record entry reaches `bodyEditRecordWarning` with every other condition true. Dropping the `isDone` guard would make the row warn and fail it. The missing-record-file row does the same for `recordStored`, because `os.Remove` leaves the entry discussed and stamped. The gate also passed with both rows in.

I found no problem with the other three proposals.
- F2 is a one-word doc rename.
- F3 text matches the guard conditions: open entry, record stored, stamp differing from the new body, and a warning that names `--record-file` and `lym loom decision add`.
- F4 asserts only that the `warning` key is absent in the `--record-file` success path, which the review premise supports.

Tool calls:
1. Read `board.go:205-222` and grepped for code that clears `Discussed` or the stamp, because my context lacked what `SetStatus` does to a discussed entry.
