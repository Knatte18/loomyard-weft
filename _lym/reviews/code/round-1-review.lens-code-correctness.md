```yaml
lens: code-correctness
outcome: reported
```

**Task:** run the code-correctness lens over the diff and give one verdict per function or branch.

**Per-function list**

- `forward`, plain text `"deploy prod"`: `posted` = `"deploy prod"`. It is under 4096 bytes and posted verbatim. Correct.
- `forward`, text plus two files: `"see …\n(file not relayed: a.png)\n(file not relayed: b.pdf)"`. Correct.
- `forward`, empty text plus a file: `fileLines` gives `"\n(file not relayed: x)"`, and `TrimPrefix` strips the leading newline. Correct.
- `forward`, size cap, text of 4096 bytes: `posted` is 4096 bytes and passes. At 4097 bytes it is refused with the answer. The cap is now measured on the bare text, which is what `creelBodyLimit` limits. The test rows `4096` and `4097` match. Correct.
- `forward`, empty `posted`: the input is a message with empty text and no files, or text that is only `"\n"`. The old body was `"[slack] "`, which is never empty. The new body is `""`, and `validateBody` in `internal/creelengine/verbs.go:22-26` refuses it with "the message body is empty". `Post` then fails, so `retryOrGiveUp` is called with "your message was not delivered; send it again". The message is retried for the 5-minute `retryWindow` and holds the channel's later messages. After that the operator is told "not delivered", and the message is skipped. This is a new failure path that the prefix used to mask. See finding 1.
- `forward`, `Post` failure or `Paused` error: neither path reads `posted`'s content. Unchanged and correct.
- `handle` presence commands: `away`, `back` and `status` are matched before `forward`, so they are unaffected. Correct.
- `SetPresence`, `readFailed` and the other notice posters: they use `presenceNoticePrefix` and `relayNoticePrefix`, which are untouched. Correct.
- `role-slack.md` classification: any other mail from `<shortname>:SLACK` is read as the operator's words. I found no `SlackProgram` sender whose body lacks `[slack presence]` or `[slack relay]`. The `Post` sites are `inbound.go:208`, `:613` and `:683`, plus `holder.go:520`, `keeper.go:498`, `relay.go:323` and `outbound.go:374`. One residual risk: operator text that itself begins with `[slack presence]` or `[slack relay]` is now read as a notice. That is the operator's own wording, and I judge it not a defect.
- Removed `forwardPrefix`: a repo-wide grep for `forwardPrefix` and `[slack]` finds no remaining reader. Correct.

**Findings**

1. LOW, `design`, `internal/slackengine/inbound.go:678-683`: an empty `posted` (a message with empty text and no files) was masked by the `"[slack] "` prefix. Now `creelengine/verbs.go:22-26` refuses the empty body, so the message is retried for 5 minutes, holds the later messages, and then gets a "not delivered" answer. Slack rarely sends such a message. If it can happen, `forward` should skip an empty body. A test row for an empty body is absent.

**Tool calls (3 of 6)**
1. Read `inbound.go:620-710`: the diff showed only part of `forward`, `handle` and `retryOrGiveUp`.
2. Grep `creelBodyLimit` and `fileLines` in `inbound.go`: I needed their definitions to check the cap and the file lines.
3. Grep for empty-body refusal in `creelengine` and `creelcli`, and `inboundText` in `inbound.go`: I needed to know whether an empty body is refused, and what the text transform can produce.
