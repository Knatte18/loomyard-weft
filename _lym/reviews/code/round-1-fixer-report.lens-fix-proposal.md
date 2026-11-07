```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F4) and report where one is wrong or leaves a gap.

**LOW — `outbound_test.go`, `OutboxFileWritableAgainIsWrittenFromTheRecordHeldInMemory` (F2 recovery row).**
- Summary: the row passes with `out.unsaved = nil` at `outbound.go:266` deleted, so it does not pin the clearing the review asked it to pin.
- Evidence:
  - Without the clear, read 2 still loads the stale in-memory record, posts messages 2 and 3, and writes the file through `save`.
  - The file then holds `handled` as the test expects, so the test passes with the defect in place.
  - The clear only shows when a later read must take the file's content instead of memory, and no read follows read 2.
- Proposed change that would pin it: add a third read after the recovery. Between read 2 and read 3, rewrite the outbox file with a record that differs from memory, or remove it. Then assert that read 3 posts from the file's state.

**LOW — `docs/overview.md:870` (F3 proposal as applied).**
- Summary: the proposed wording, "a message Slack accepted is not posted again", is unbounded, while F4 bounds the same promise to one relay process.
- Evidence:
  - `doc.go:45` states the same sentence but is followed at line 49 by "lost when the relay process ends, so a restart ... can post that mail again".
  - The overview line sits alone in a bullet and carries no such limit.
  - A reader of the overview alone gets the stronger, false promise.
- Proposed change: end the overview line with "…is not posted again within one relay process".

Everything else holds:
- **F1:** the new stale-line test fails if the line reposts. Read 1 clears `Stale` in the in-memory record, and read 2 loads that record, so a regression that lets the line repost would show up as a second count line.
- **F2, joined fault and budget:** both rows fail if the `errors.Join` or the `saveFault` return is dropped.
- **F4 header:** "within one relay process" matches `out.unsaved`, which lives in the `outbound` struct.

Further tool calls: none, because the context already held every changed file.
