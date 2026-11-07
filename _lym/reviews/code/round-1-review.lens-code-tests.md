```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none.

| Behaviour | Pinning test | Notes |
|---|---|---|
| `chat.postMessage` with `channel` as a string decodes and returns nil | `TestAPI/EachMethodSendsItsRequestAndDecodesSlacksDocumentedAnswer` (api_test.go:182, `PostMessage` at :205, body at :161) | Fails on the old shared `response` type. |
| `conversations.info` decodes the `channel` object | Same test (:195-201) | Asserts `ID`, `Name`, `IsMember`. `IsPrivate` is false in the fixture, so a wrong tag on it would go unnoticed (not a finding). |
| `conversations.history` decodes `messages`, `has_more` and `next_cursor` | Same test (:208-229) | Asserts the decoded messages and the cursor follow-up request. |
| `auth.test` and `users.info` read nothing past `ok` | Same test (:192, :202) | Documented bodies are used. |
| `call` returns the body once `ok` is true, and reads only the envelope | Same test | Indirectly covered through the methods above. |
| `call` error mapping (429, ratelimited, APIError, 5xx) is unchanged | `TestAPI/FailureKinds` (:282) | Unchanged behaviour, already covered. |
| `decodeAnswer` fails with a plain error on an unreadable answer | none | Not a gap: the error text is the same as before the diff, and the old code had no row for it either. |
| `History` and `Latest` go through the new `history` helper | `HistoryFollowsThePagesAndReturnsOldestFirst` (:244) and `LatestOnAnEmptyChannelFindsNothing` (:270) | Covered. |
| A failed outbox write after an accepted post does not repost, with the acknowledgement confirmed | `TestOutboundRead/OutboxWriteFailingAfterAnAcceptedPostAcknowledgesTheMailAndNeverRepostsIt/AcknowledgementConfirmed` (outbound_test.go:260-295) | `onPost` (inbound_test.go:332) turns the outbox path into a directory, so every write fails. The old code returned the write error from `post`, so creel released the read and the message went out again. The test asserts one answer and one ack. |
| The same, with the acknowledgement refused and the batch redelivered | The `AcknowledgementRefusedAndTheBatchRedelivered` row | Exercises the in-memory `unsaved` record in `loadOutbox`: the second read must skip message 1, and it does (one answer). |
| `read` returns `saveFault` on the `err == nil` path | Both rows assert the `outbox file` error | Covered. |
| `saveFault` returns on the `errReadBudget` path | none | No test makes the budget end while a write fails. The two combine only through `save` plus `read`'s switch (outbound.go:92-93). |
| `saveFault` is joined into the error on the failed-post and rate-limit paths | none | The `errors.Join(err, out.saveFault)` at outbound.go:97 is untested, and so is the rate-limit-plus-write-failure combination. |
| `saveFault` resets at the start of each `read` | Indirect only | The second `read` of the redelivered row reports a fault from its own writes, so a missing reset would not fail it. Weak. |
| `unsaved` clears after a later successful write | none | Every row leaves the directory in place, so a successful `save` after a failed one (outbound.go:266) is never run. |
| `refused` saves without returning the write error | none | The `refusalCap` tests use a healthy outbox. |
| `reportStale` with a failing write posts the stale line once | none | The removed rule "the line waits after a failed write" has no replacement test. Both rows use fresh mail and `Stale` stays 0. |
| `cloneOutbox` isolates slices and maps | none | A shared-slice bug would show only through aliasing. It is an internal helper, so a separate test would be a per-helper test; not raised. |

**Findings**

1. **MEDIUM, design (test gap).** The stale-mail line has no failing-write case. Location: outbound_test.go:260 for the test, outbound.go:120-123 and :143-144 for the code. The diff deletes `errOutboxWrite` and the rule that the line waits after a failed write. The new test never gives a stale count, so a regression that reposts the line would pass. Evidence: both rows send one fresh message from `mailFrom` and nothing older. Fix: add a row (or a `staleMail` flag) that sends old mail plus fresh mail under the failing outbox, and assert one count line over two reads.

2. **LOW, design (test gap).** The paths where `saveFault` rides along with a non-nil `err` are untested: the `errors.Join` at outbound.go:97, the `errReadBudget` return at :93, and the `unsaved` reset at :266. Evidence: no test combines a failing write with a failed post, a rate limit or the posting budget, and none restores the outbox file before a later read. Fix: add rows to the same table-driven test, for example `onPost` that breaks the outbox plus a second message that fails to post, and a row that removes the directory before the next read and asserts `Read` returns nil.

3. **NIT, design (test economy).** No redundant tests found. The new `TestAPI` subtest overlaps `HistoryFollowsThePages…` on pagination but pins the documented body shapes, so it is not a duplicate.

Tool calls (1):
- grep of `inbound_test.go` for `onPost`, `ackErr` and the fake `ReadWait` and `PostMessage`: the manifest did not hold the harness, and I needed it to confirm the failure injection and the redelivery mechanics.
