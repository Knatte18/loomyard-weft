```yaml
lens: fix-affected-set
outcome: reported
```

The lens is fix-affected-set: rerun the search behind each set-premise finding and check that the fix covers every member of the set.

**Unchecked:** F6, F8 and F9 have no set premise. F4 and F5 were checked from context only, with no further search. Their members are all covered: F4's four harnesses and `runRelay` all set `LiveHubs`, and F5's three holder branches each have a row now.

**Checked:**
- **F1:** the `WriteToken`, `ReadToken` and `RequireConfigured` sites are all covered by the new card 5's Edit list. Its `TestSlackCLI` "rows that call" clause covers the `Setup` and refusal rows.
- **F2:** every post site without recovery is covered: `outbound.go:141` and `:197` by card 12, `inbound.go:424` by card 11.
- **F3:** every site naming `groups:history` is covered. These are the `botScopes` text, the `loginCmd` help at `cli.go:204` (card 6 edits it), and the `cli_test.go:481` row (card 6 test paragraph).
- **F7:** two members are missed, below.

Findings:

1. **LOW** — card 13's Docs paragraph, the new `internal/slackengine/doc.go` sentences.
   The list of sentences to drop leaves out the other single-channel sentences.
   Evidence: `internal/slackengine/doc.go:10` reads "the setup file with the channel and the operator (`Setup`)". Card 7 cuts setup down to the operator, so this line goes stale.
   `doc.go:44` reads "posts mail from the hub's orchs and programs to the channel", which card 12 makes false.
   Neither is in the drop list, so they can survive the same way the review's F7 premise describes.

2. **LOW** — card 13's Docs paragraph, the new `internal/slackcli/doc.go` sentence.
   It drops only `setup` recording the channel, and leaves the one-token login sentences.
   Evidence: `internal/slackcli/doc.go:5` reads "`login` stores the machine's bot token". `doc.go:9` reads "`login` reads the token from standard input", and its "reads one line with echo off" contradicts card 6's two hidden lines.

Tool calls:
- One grep for `PostMessage(` in non-test `internal/slackengine` files and for `groups:history`/`botScopes` across the repo. The post sites and the scope sites were not in my context as a full set.
