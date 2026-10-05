# Card 4 — compact-seam

**Edit:**
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/session_test.go`

**Intent:**
Give the shuttle seam a compact command, so the `/compact` text lives only in `claudeengine` (PATTERN-shuttle-provider-seam).

`SessionCycler` gains `CompactSessionSequence(focus string) []PaneInput`, the key choreography that compacts the live session's context keeping what focus names.
`Runner.CompactSession(guid, focus string) error` mirrors `Runner.ClearSession`: same capability check, run lookup and live-strand check, then plays the sequence; it refuses a focus containing a newline, since it is typed as one line.
Claude's `CompactSessionSequence` returns `/compact <focus>` typed and submitted with no leading Escape, for the same reason its clear sequence has none; with an empty focus it types bare `/compact`.
Update the `session.go` header comments in both packages and the `SessionCycler` doc to name compaction.

Tests: `claudeengine/session_test.go` asserts the exact sequence for a focus and for an empty one;
`shuttleengine/session_test.go`'s `cyclerEngine` gains the method, and a test asserts `CompactSession` plays it into the strand, refuses a multi-line focus without playing anything, and errors naming the capability when the engine lacks it.

**ImpactSummary:** Adds one method to the SessionCycler interface and its single production implementer (Claude) plus one Runner method; existing callers are unaffected.
