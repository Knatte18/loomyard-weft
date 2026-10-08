# Card 5 — claude-session-usage

**Create:**
- `plan:internal/shuttleengine/claudeengine#Claude.SessionUsage` -> `func (c *Claude) SessionUsage(sessionID, workdir string) shuttleengine.SessionUsage`
- `plan:internal/shuttleengine/claudeengine#transcriptTokens` -> `func transcriptTokens(path string) (fresh, cacheRead int, err error)`

**Edit:**
- `internal/shuttleengine/claudeengine/usage_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#claudeProjectDirFor`

**Intent:**
Claude's implementation of card 4's `UsageReader`, in `usage.go`, so every transcript-shape detail stays in `claudeengine` (`PATTERN-shuttle-provider-seam`).
`SessionUsage` finds the parent transcript `<project dir>/<session id>.jsonl` and the fork transcripts under `<project dir>/<session id>/subagents/`, the layout the fork audit already reads, through `claudeProjectDirFor`.
`transcriptTokens` sums one transcript's main-chain assistant usage: fresh is input plus cache-creation plus output tokens, cache-read is read tokens.
Claude Code writes one transcript line per content block of an assistant message, each carrying that message's usage, so the sum counts each `message.id` once;
malformed lines and sidechain entries are skipped.
The parent's totals include its forks, which are also broken out as the fork count and the forks' fresh and cache-read sums.
An unreadable parent transcript or fork transcript makes the whole reading unknown, never an error, since the layout is a Claude Code internal;
a missing `subagents/` directory is zero forks.
A compile-time assertion pins that `*Claude` implements `shuttleengine.UsageReader`.

`usage_test.go` gains tests over fixture transcripts under a temporary `HOME`: a parent with repeated message lines counted once, two forks summed and broken out, no `subagents/` directory, and an unreadable transcript reading unknown.
The `claudeengine` package doc describes the session-usage reading beside the context reading.

**ImpactSummary:** Adds one method and one helper to claudeengine and their tests; no existing claudeengine member changes.
