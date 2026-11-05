```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set (F1–F5) for sibling and cross-reference agreement, through the fix-consistency lens.

**LOW** — `internal/slackengine/doc.go:26`
- Changed text it concerns: `outboxRecord` (`internal/slackengine/outbound.go`) now also persists the stale-mail count in `Stale`.
- Summary: the package doc still says the outbox record "keeps each message already posted until creel confirms the acknowledgement". It does not mention the persisted stale count.
- Evidence: the doc comment at `doc.go:26` has that wording. The record has held refusal counts since before this round, so the sentence was already incomplete. F1 adds a second counter that the doc omits, and the `outboxRecord` struct comment is the only place that names it.
- This is only a stale cross-reference. No behaviour disagrees.

I found no other inconsistencies:
- The `read` doc comment ("the outbox record keeps each message already handled until then") still holds.
- The success path `writeOutbox(outboxRecord{})` clears `Stale` along with the rest of the record. That matches the F1 fix's "clear the count after a confirmed acknowledgement".
- The budget path, the Slack-failure path and the write-error path all reach `reportStale` through `post`, so no return path was left with its old behaviour.
- The F5 revert restores the original wording. `TestConfigLong` asserts only `EDITOR`, `VISUAL`, `code --wait`, `nano`, `--set` and `--layer`, so it holds either way.
- The F4 table row keeps the old row's assertions.
- The F2 and F3 test names and fixtures match their neighbours' conventions (`newHub`, `seedReedState`, `pauseRecordPath`-style setup).

Tool calls:
1. `grep -rniE "outbox|older than 24|stale mail"` over `*.md` and `doc.go`, excluding the review and `.lym` directories. My context held the Go files but no docs, so I needed to look for twin descriptions of the outbox record and the stale line.
