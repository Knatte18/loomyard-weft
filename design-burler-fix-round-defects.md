# Burler fix round: rerun reason and fork git in its own scratch repo

`burler-fix-round-defects` · Task · bug, burler, shuttle

Depends on: [`webster-parent-calls-flow`](design-webster-parent-calls-flow.md), [`halt-escalation-messages`](design-halt-escalation-messages.md)

{"error":"invalid json: invalid character 'b' looking for beginning of value","ok":false}

## Triage against main (2026-11-21)

- Rerun: `internal/shedadapters/burlerfix.go` renames a leftover round fixer report to a timestamped name before each attempt; the fixer prompt never says why the step runs again.
- Fork git: 27498365 lets a fork run git in a repository it created, but `internal/burlerengine/cluster.go` still refuses every push, and the halt text still says revert.
