```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for agreement with its surroundings (lens fix-consistency).

Findings:

- **LOW** — `docs/overview.md:870` and `internal/slackengine/doc.go:45`.
  - The fix for F3 and the fix for F4 disagree.
  - `doc.go` now says the in-memory record is lost on a relay restart, and the `outbound.go` header says "within one relay process".
  - The overview line says "a message Slack accepted is not posted again" with no restart bound.
  - That is the same unbounded wording F4 called out in `doc.go`, and the reviewer's own `fix` text for F3 asked for exactly this phrase.
  - It is the one remaining twin of the guarantee that lacks the bound. A reader of the overview would not learn that a restart can repost.
  - If the overview should match `doc.go`, add a short bound there, for example "within one relay process".

Everything else agrees:
- `outbound.go` line 71 ("a redelivered read posts nothing twice") and line 57 are scoped to a read or an in-memory record, so they are consistent.
- The `orchcli/start.go:294` hit is unrelated.
- The new test names match the behaviors the `doc.go` and header text describe.

Tool calls made:
1. One grep for other statements of the outbound guarantee across `.go` and `.md` files, because my context only held the files the fixer read.
