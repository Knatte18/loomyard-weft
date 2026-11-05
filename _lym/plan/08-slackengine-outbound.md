# Card 8 — slackengine-outbound

**Create:**
- `plan:internal/slackengine#outbound` -> `type outbound struct`
- `plan:internal/slackengine#outbound.read` -> `func (out *outbound) read(ctx context.Context) (time.Duration, error)`
- `plan:internal/slackengine#outboxRecord` -> `type outboxRecord struct`
- `plan:internal/slackengine#outboundText` -> `func outboundText(shortname string, msg creelengine.Message) (string, bool)`
- `internal/slackengine/outbound_test.go`
- `plan:internal/slackengine#TestOutboundRead` -> `func TestOutboundRead(t *testing.T)`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `plan:internal/slackengine#Mailbox`
- `plan:internal/slackengine#SlackClient`
- `plan:internal/slackengine#Clock`
- `internal/creelengine#IsOrchStrand`
- `internal/agentname#TaskSlug`
- `internal/creelengine#Message`
- `internal/creelengine#ErrWaitTimeout`

**Intent:** Add one outbound read of the relay's own mailbox, posting orch and program mail for `<shortname>:SLACK` to the channel at least once and never twice for one acknowledged read (D5).

`outbound.read` calls `Mailbox.ReadWait` with a bounded wait and a handler, and keeps `outboxRecord{Handled []int64; Refusals map[int64]int}` at `OutboxPath` through `internal/state`:
- The handler goes through the messages oldest first and skips each id the record holds.
- A message creel stamped (`Posted`) more than 24 hours before the clock's now is added to the record unposted and counted.
- A message whose sender is neither an orch strand (`creelengine.IsOrchStrand`) nor a program of the hub other than `<shortname>:EXTERNAL` is dropped, logged at Warn and added to the record.
- Any other message is posted with `PostMessage` as `outboundText` renders it, then added to the record; the record write follows each post, so a crash between them reposts that message once (D5's bound).
- After the last message, one line naming how many were skipped as older than 24 hours and `lym creel log --to <shortname>:SLACK` is posted when any were.
- The handler stops posting 25 seconds after it began, inside creel's 30-second acknowledgement window (`ackReadTimeout` in `internal/creelengine/server.go`), and returns a release error with the rest unposted; a failed post returns its error at once. Either way the read is released unacknowledged, and `read` returns the wait before the next read: the floor wait, or a `*RateLimitError`'s `RetryAfter`.
- An `*APIError` adds one to that message's refusal count in the record; at the fifth, the handler posts one line naming the message's id and sender and `lym creel log --to <shortname>:SLACK` and adds the message to the record as handled, whether that line posts or not. A transport failure, a `5xx` and a `*RateLimitError` count toward no cap.
- When `ReadWait` returns nil, the daemon confirmed the acknowledgement, and the record is cleared; any error keeps it, so the next read skips what it holds.
- `read` records whether its handler ran, since `ReadWait`'s one timeout spans the wait, the handler and the confirmation (card 3).
  A bounded wait that ends with no mail, `ReadWait` returning `creelengine.ErrWaitTimeout` with the handler not run, is an empty read, not a failure: nothing is logged, the record stays as it is, and `read` returns the doubled idle wait.
  An `ErrWaitTimeout` after the handler ran is a lost confirmation, not an empty read: the record stays, and `read` returns the floor wait, as after any error that keeps the record, whether or not a post succeeded, since mail arrived.
- A successful post makes `read` return the floor wait, so the relay returns its wait to the floor (D4's activity).

`outboundText` renders `[<sender>] <body>` with `&`, `<` and `>` escaped, and, for a sender carrying a beam slug (`<shortname>:<slug>:orch` or `<shortname>:<slug>:<PROGRAM>`, read through `agentname`), ends with `(answer: @<slug> …)`; it answers false for a sender it does not post.

`internal/slackengine/outbound_test.go` is tier0, and uses the in-memory `Mailbox`, with its scripted reads and recorded acknowledgements, `SlackClient` and `Clock` of `inbound_test.go` unchanged:
`TestOutboundRead` rows — mail from the prime orch, from a program and a reply to a `[slack]` message is posted once and the read acknowledged; the prime orch's mail is posted as `[ly:orch] a &amp; &lt;b&gt;` with no hint, and a beam orch's and a beam program's posted text ends with `(answer: @<slug> …)`; mail from a non-orch strand and from `<shortname>:EXTERNAL` is not posted and is acknowledged; a Slack failure on the second message of a read, then success on the next read, posts each message exactly once; a message Slack refuses 5 times with `ok: false` is replaced by one line naming its id, its sender and `lym creel log --to <shortname>:SLACK`, and the read is acknowledged; a transport failure, and a `5xx`, each keep the message queued past 5 tries; a backlog whose posting outlasts 25 seconds on the fake clock releases the read and posts each message exactly once across the next reads; mail stamped more than 24 hours before the read is acknowledged unposted with one line naming the count and the log command; a refused acknowledgement keeps the record and the next read reposts nothing it holds; a `429` returns its `Retry-After` as the wait; a wait that ends with `ErrWaitTimeout` and no handler run logs nothing, keeps the record and returns a longer wait than the floor; a fake `ReadWait` that runs the handler, which posts, and then returns `ErrWaitTimeout` keeps the record and returns the floor, as does one whose handler posts nothing, every message already in the record.

**ImpactSummary:** New unexported code inside the new package; it reads creel's message type and calls nothing outside the package.
