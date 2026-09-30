# Card 2 — claude-context-usage

**Create:**
- `internal/shuttleengine/claudeengine/usage.go`
- `internal/shuttleengine/claudeengine/usage_test.go`
- `internal/shuttleengine/claudeengine/testdata/usage-main-chain.jsonl`
- `internal/shuttleengine/claudeengine/testdata/usage-sidechain-last.jsonl`
- `internal/shuttleengine/claudeengine/testdata/usage-after-clear.jsonl`
- `internal/shuttleengine/claudeengine/testdata/usage-malformed.jsonl`

**Uses:**
- `internal/shuttleengine/claudeengine/events.go`
- `internal/shuttleengine/claudeengine/background.go`
- `internal/shuttleengine/session.go`

**Intent:**
Implement `SessionCycler.ContextTokens` for Claude, per decision D7: the context a session holds is read from its own transcript, located through the Stop payload rather than guessed from directory listings.

`func (c *Claude) ContextTokens(turnEnd shuttleengine.Event) (int, bool)` decodes `turnEnd.Raw`, takes its `transcript_path`, reads that file, and hands the bytes to an unexported `transcriptContextTokens(data []byte) (int, bool)`.
Following the transcript through `/clear` needs no state: a clear starts a new session ID and transcript file, and each later Stop payload names the new file, so reading the path from the turn end being evaluated is the whole mechanism.

`transcriptContextTokens` scans the JSONL and keeps the last entry that is a main-chain assistant message: `type` is `"assistant"`, `isSidechain` is not true, and `message.usage` is present.
Its reading is `input_tokens + cache_creation_input_tokens + cache_read_input_tokens` of that entry.
An entry whose three fields sum to zero is skipped, since a synthetic assistant entry (an API error, an interrupt) carries zero usage and says nothing about the context.
Malformed lines are skipped.
No qualifying entry, a missing `transcript_path`, and an unreadable file all return `(0, false)` — the parser degrades to "usage unknown" and never errors, per the decision record's open risk on Claude Code internals.
All payload and transcript shape knowledge stays in this file, per the Shuttle Provider-Seam Invariant.

The fixtures are transcript excerpts in Claude Code's JSONL shape:
`usage-main-chain.jsonl` has several main-chain assistant entries with different usage, the last one the expected reading, followed by user and tool-result lines;
`usage-sidechain-last.jsonl` ends in sidechain assistant entries with larger usage that must be ignored, plus a trailing zero-usage synthetic entry;
`usage-after-clear.jsonl` is a short second transcript standing in for the file a post-clear Stop names;
`usage-malformed.jsonl` holds only malformed lines and non-assistant entries.

`usage_test.go` covers: the sum from the last main-chain assistant entry;
sidechain and zero-usage entries ignored;
two Stop events whose `transcript_path` names `usage-main-chain.jsonl` and then `usage-after-clear.jsonl`, each reading its own file;
and `(0, false)` for the malformed fixture, for a missing file, and for a Stop payload with no `transcript_path`.
Each test writes its Stop payload's `Raw` pointing at the fixture's absolute path.
