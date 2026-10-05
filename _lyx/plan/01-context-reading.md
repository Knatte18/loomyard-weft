# Card 1 — context-reading

**Edit:**
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/usage.go`
- `internal/shuttleengine/claudeengine/usage_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchcli/session.go`

**Create:**
- `internal/shuttleengine/claudeengine/testdata/usage-boundary-last.jsonl`
- `internal/shuttleengine/claudeengine/testdata/usage-assistant-after-boundary.jsonl`

**Uses:**
- `internal/shuttleengine/claudeengine/testdata/usage-main-chain.jsonl`
- `internal/shuttleengine/claudeengine/testdata/usage-sidechain-last.jsonl`

**Intent:**
Make a compaction boundary a context reading (D1) and read the transcript from its end (D3).

In `internal/shuttleengine/engine.go` add a provider-neutral `ContextReading` struct: `Tokens int`, `Known bool`, `Compacted bool` (the reading came from a compaction boundary) and `BoundaryAt time.Time` (that boundary's timestamp, zero otherwise).
`SessionCycler.ContextTokens` returns `ContextReading` instead of `(int, bool)`, and `Runner.ContextTokens` in `internal/shuttleengine/session.go` returns `(ContextReading, error)`.
A reading with `Known` false still never counts as over any threshold; say so on the type.

In `internal/shuttleengine/claudeengine/usage.go` replace the whole-file `os.ReadFile` and forward scan with a backward reader over the transcript named by the turn end's `transcript_path`.
It reads the file from the end in growing chunks (start small, double each round) until it has found the latest qualifying entry of each kind or reached the file start, parsing only complete lines, so an entry spanning a chunk edge is read whole once the next chunk supplies its start.
The two kinds are the latest main-chain assistant usage entry (today's rules: not sidechain, non-zero input + cache-creation + cache-read) and the latest compaction-boundary entry: `"type":"system"`, `"subtype":"compact_boundary"`, not sidechain, whose `compactMetadata.postTokens` is the post-compaction count and whose top-level `timestamp` (RFC 3339) is the boundary time.
Whichever of the two sits later in the file is the reading; a boundary reading sets `Compacted` and `BoundaryAt`.
Keep the core testable as a function over an `io.ReaderAt` plus size and an initial chunk size, so a test can force a tiny chunk.
Every failure still degrades to an unknown reading and never errors; every transcript shape fact stays in this file (PATTERN-shuttle-provider-seam).
Update the file's header comment and `internal/shuttleengine/claudeengine/doc.go` where they describe the reader.

Reader tests in `usage_test.go`, with the two new fixtures:
a boundary after the last assistant entry yields the boundary's `postTokens`, `Compacted` true and its timestamp;
an assistant entry after the boundary yields the assistant usage with `Compacted` false;
sidechain and zero-usage entries are ignored (existing fixtures);
a qualifying entry spanning a chunk edge is read whole (tiny initial chunk);
a file with neither kind yields unknown.
`session_test.go`'s `cyclerEngine` fake and assertions move to the new return shape.

Adapt the callers so the tree builds with unchanged behaviour: `orchengine.Session.ContextTokens` in `internal/orchengine/watcher.go` returns `(shuttleengine.ContextReading, error)`, the watcher stores `Tokens`/`Known` exactly as today, `internal/orchcli/session.go`'s `runnerSession` delegates, and `watcher_test.go`'s `fakeSession.ContextTokens` returns a `ContextReading` built from its `usage` map.

**ImpactSummary:** Changes the SessionCycler and Runner context-reading signature and its two consumers (orchengine's Session seam and orchcli's adapter); behaviour outside the reader is unchanged.
