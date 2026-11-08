```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff changes, I name the test that pins it.

**Per behaviour**
- **Body forwarded without the prefix, channel and thread reads.** Pinned by `TestInboundRead` in `inbound_test.go`, in the subtests for forwarding, `ResyncAndStartRead…`, `CursorForwardsEachMessageOnce…`, `ThreadCursorOlderThanADay…` and the thread subtests. All use `wantPosts` with exact bodies. Reintroducing the prefix fails every row.
- **Text with no marker of its own, so `@payments x` stays verbatim.** Pinned by `wantPosts(mail{primeOrch, "@payments x"})`.
- **File lines appended after the text.** Pinned by the row expecting `"see https://x.io/a\n(file not relayed: a.png)\n(file not relayed: b.pdf)"`.
- **Slack markup converted to plain text.** Pinned by `SlackMarkupReachesTheOrchAsPlainText`.
- **Size cap now counts the text alone.** Pinned by the `AtTheCap` row (4096, posted) and the `OverTheCap` row (4097, refused). Both rows lose their `len("[slack] ")` offset. The two rows sit exactly on `creelBodyLimit`, so a reintroduced prefix fails `AtTheCap`.
- **Presence and relay notices stay prefixed.** Pinned by the `[slack presence]` rows in `inbound_test.go` and `slackcli/cli_test.go:878`, and by the `[slack relay]` helpers (`relayNotices`, keeper, holder). All unchanged and still valid.
- **Relay-level forward.** Pinned by `relayRun.forwarded` in `relay_test.go`, now matching `posted.Body == text`.
- **Duplicate tests:** none. The diff adds no test and only edits expectations.
- **Test economy:** the edits add no assertion on prose.

**Findings**

1. **LOW, design, `internal/slackengine/inbound.go:677-678`, `inbound_test.go` (no row).**
   - The body is now exactly the trimmed text plus file lines, so a message with empty text and no files would post an empty body.
   - Before the diff that case posted `"[slack] "`, which was never empty. No test row covers an empty forwarded body.
   - A Slack message with an empty `text` and no files is unlikely, since a `file_share` carries file lines.
   - If creel refuses an empty body, `handle` returns the error and holds the cursor for the 5-minute retry window. I did not check how the mailbox treats an empty body.
   - Add a row if the empty case is reachable. Otherwise this is a nit.

**Tool calls**
1. `grep` in `inbound.go` for empty-text checks. My context did not show whether `handle` or `forward` guards an empty text.
2. `Read` of `inbound.go` lines 624-643. My context did not show the body of `handle`, which feeds `forward`.
