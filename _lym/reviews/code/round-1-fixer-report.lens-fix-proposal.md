```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F5). I made no further tool calls.

**LOW — F1, `internal/battencli/dobbyreads.go`, the landed-commit read closure.**
- The proposal warns and returns nil on any read error.
- A done run stays in the listing, so a permanently unreadable landed-commit file, such as a directory or a permission error, logs the same `logger.Warn` on every tick. Nothing dedups it.
- The sibling Warns in `dobbyParentRuns` have the same exposure, so this matches existing practice.
- A single transient I/O error on the tick that first shows the run past Land settles the notice as "landed, commit unknown", and that notice is never resent. The reviewer accepted this loss of precision (D6), so it is a trade-off, not a defect.

**No findings** on the other four.
- **F2:** the new first-try test asserts the order `merge,mark,record landed1,push`. Deleting the first-try `recordLanding` call drops `record landed1`, so the test fails with the defect in place.
- **F3:** two of the new rows are sound, the withdrawn-request and raised-request rows. They fail if the later reading does not replace the earlier one.
- **F3:** the failed-board-read row is weaker. In `landNoticeRequests`, a failed read returns `nil, false`, so `requests[slug]` is never on the board and the `tasksRead &&` guard in `landNotices` is redundant. The row would not catch removing that guard. It does catch code that resets the memory on a failed read, which is the behaviour the review named.
- **F4:** the new `TestEntryDates` step and the `TestFabricReplay` row each fail if `replayFields` drops the field's row or the field moves the revised date.
- **F5:** the two wiring tests use the real hub fixtures and passed the tier1 gate. I found no case where the proposed tests pass with the wiring broken.

I made no further tool calls: the context already held the review, the changed files and the gate results.
