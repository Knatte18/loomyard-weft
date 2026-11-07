```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens over the changed code (the three new tests and the comment edits).

**Findings**

1. **MEDIUM**, `outbound_test.go`, subtest `OutboxFileWritableAgainIsWrittenFromTheRecordHeldInMemory`. The test passes with the defect it is meant to catch left in place.
   - The finding F2 asked for was `unsaved` clearing after a later good write, which is `out.unsaved = nil` at `outbound.go:266`.
   - Take that line out and read 2 still behaves the same:
     - `save` writes the file before it reaches that line, so the file holds `handled` either way.
     - `loadOutbox` reads `unsaved` at the start of read 2, which is before any good write.
     - `saveFault` is reset at the start of each read, so read 2 still returns nil.
   - Only a third read would show the stale `unsaved` standing in for the file. No read after read 2 exists.
   - The name and comment also overstate what the test shows. At the end of read 1 the final `save(outboxRecord{})` fails, so the record held in memory is empty. The file read 2 writes holds only messages 2 and 3, not message 1 from the failed write.
   - To make the test fail with the defect, run a third read after the file has recovered, with a batch that holds a message not yet handled. Make that read also fail its acknowledgement or end on the budget, so the file is left holding the record. Assert the file holds that message. With `unsaved` still set, `loadOutbox` would return the old in-memory record and drop the file's content, so the assertion would fail.

2. **No finding.** The other two new tests fail with their defects in place. I ran each step by step.
   - `StaleMailLineIsPostedOnceWhenTheOutboxWriteFails`:
     - Read 1 posts the fresh mail, and the directory is created at that post.
     - The count line posts once, and read 1 returns the refused acknowledgement joined with `saveFault`.
     - Read 2 loads `unsaved` with `Stale` at 0 and posts nothing.
     - Without the in-memory record, read 2 would load the empty file, recount both old messages and post a second line. `lines != 1` would catch that.
   - `OutboxWriteFailureRidesAlongWithAFailedPostAndWithTheEndedBudget`:
     - `FailedPost`: message 1 posts and its write fails, then message 2's post fails with a plain error. `errors.Join` at `outbound.go:97` carries both errors, and removing it drops "outbox file" from the error.
     - `PostingBudgetEnded`: the posts advance the clock to 15s and then 30s, so message 3 hits `errReadBudget`. That branch returns `saveFault`, and returning nil would fail the "outbox file" check.

3. **No finding.**
   - The header edit in `outbound.go` and the sentence added to `doc.go` are consistent with each other and with the code.
   - The edit to `docs/overview.md` matches `doc.go`.
   - No signature changed, so no caller is affected.

**Calls made:** none. My context already held the changed files and the gate result.
