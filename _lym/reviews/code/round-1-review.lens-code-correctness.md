```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each changed function or branch on a concrete case. I made no tool calls.

**Per function or branch: case run, verdict**

- `api.go call`:
  - Case: 200 with `{"ok":true,"channel":"C1"}`. The envelope decodes, `ok` is true, and the body is returned. Correct.
  - Case: 200 with `{"ok":false,"error":"ratelimited"}`. It returns a `*RateLimitError`. Correct.
  - Case: 200 with an empty body or non-JSON. `decodeAnswer` returns a plain error. Correct, and the same as before.
  - Case: 200 with `null`. The envelope is zero, so it returns an `*APIError` with an empty code. Correct, and the same as before.
- `api.go decodeAnswer`: a decode failure is wrapped with the method name, and the zero `T` is returned alongside the error. Correct.
- `api.go AuthTest`, `UserInfo`, `PostMessage`: they read only `ok`, so the documented `chat.postMessage` body (`channel` as a string) can no longer fail to decode. Correct.
- `api.go ConversationInfo`: the documented object decodes into `ChannelInfo`, and extra fields are ignored. Correct.
- `api.go history`, `History`, `Latest`:
  - Case: `messages` null or missing. The slice is empty, so `Latest` returns found false. Correct.
  - Case: `has_more` true with an empty cursor. The loop breaks. Correct.
  - Case: page 2 decodes in the same shape. Correct.
- `outbound.go read`:
  - Case: the handler succeeded, the acknowledgement was confirmed, and the final `save(outboxRecord{})` fails. `unsaved` holds an empty record and `saveFault` is returned with `afterActivity`. The next `loadOutbox` ignores the file's stale `Handled`, which is right because the mail was acknowledged. Correct.
  - Case: the acknowledgement was refused or lost (handler ran, error returned). The record in memory or on disk is kept, so a redelivered batch is skipped. The error is `errors.Join`ed with `saveFault`, and `errors.As` and `errors.Is` still match the rate-limit and wait-timeout errors inside it. Correct.
  - Case: `errReadBudget`. It returns `saveFault` with `afterActivity`. Correct.
  - Case: timeout with no handler run. It returns `afterIdle` and nil. A `saveFault` from an earlier read is cleared by the `out.saveFault = nil` reset. Correct.
- `outbound.go post`:
  - Case: the unreadable-file path, where `loadOutbox` errors and nothing is posted, so nothing can repost. Correct.
  - Case: a rate-limited `postErr` returns early without posting the stale line. Correct.
  - Case: a failed stale line posts nothing and is not saved, so the count stays. Correct.
- `outbound.go postMessages`:
  - Case: an accepted post followed by a failing `save`. `Handled` is appended and the in-memory record is updated, so a redelivered batch is skipped by the `Handled` check. Correct.
  - Case: the budget check runs before each unhandled message. Correct.
- `outbound.go refused`:
  - Case: a refusal below the cap. The count is saved, or kept in memory if the write fails, and the error is returned, so the count is no longer lost on a write failure. Correct.
  - Case: the cap is reached. It returns dropped, and the caller records `Handled` and saves. Correct.
- `outbound.go loadOutbox`, `save`, `cloneOutbox`:
  - Case: a failed write followed by a later successful write. `unsaved` is cleared and the file is overwritten with the full record. Correct.
  - Case: nil `Refusals` and nil `Handled`. The clones of nil are nil. Correct.
  - Case: the clones share no state with `unsaved`. Correct.

**Findings**

None at BLOCKING, MEDIUM or LOW. Two things I checked and judged not defects:
- When a mid-read write failed but the final `save(outboxRecord{})` succeeds, `read` still returns the earlier `saveFault`. State is consistent, so this only produces a log line for a real earlier fault.
- If the relay restarts while `unsaved` is set, the older file record can repost. The summary and `doc.go` already name this bound, and the code can't close it without a different store.

**Tool calls:** none made. The manifest already held every definition this lens needed.
