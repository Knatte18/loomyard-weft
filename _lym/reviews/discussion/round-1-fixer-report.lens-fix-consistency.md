```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 change set agrees with the sibling decisions, criteria and cross-references it touches.

Findings:

1. **MEDIUM**, decision-record.md D11, first bullet ("A no closes a D6 record's open pull request with a comment naming the resolver's reason and removes the record…").
   The fix makes Finalize close a pull request, but the record's `## PATTERN changes` entry for `PATTERN-github-auth` was left as it was.
   That entry still says "only PR-Review opens, Finalize merges and pair teardown closes a pull request".
   So the new PATTERN text says no part of Finalize closes a pull request, and D11 now says it does.
   The same entry already missed D3's no closing one.
   This edit grew the conflict instead of fixing that twin.

2. **LOW**, decision-record.md D12, new Bound ("a teardown of a gone task worktree … leaves the pull request open and logs a Warn naming the task branch").
   The support log's Q7 was not updated to match.
   It still reads "Every teardown closes an open pull request with a comment".

3. **LOW**, decision-record.md D10, new text ("`readAgentActivity` also counts a child status running at the `PR-Review` row … `internal/battenshed/doc.go`'s list of live waits names it").
   D10 now changes `internal/battencli/wire.go` and `internal/battenshed/doc.go`.
   The record's `## Scope` "In" list still names no change to the quiet check's live-wait read.

No further tool calls made.
