# Card 10 — compaction-boundary-turn-ends

**Create:**
- `plan:internal/shuttleengine#CompactionBoundary` -> `type CompactionBoundary struct`

**Edit:**
- `internal/shuttleengine#SessionCycler`
- `internal/shuttleengine#Runner.CompactedSince`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/usage.go`
- `internal/shuttleengine/claudeengine/usage_test.go`
- `internal/orchcli/session.go`

**Intent:** D8, provider half: the provider reports, with a compaction boundary, how many turn ends follow it in the transcript and whether the turn end the caller read is one of them, so the orch watcher can tell a fresh compaction from an old one.
Turn-end and transcript shape stay in `claudeengine`, per `PATTERN-shuttle-provider-seam`.

Declare `CompactionBoundary` in `internal/shuttleengine/engine.go`, beside `ContextReading`, with three fields:
`At time.Time` (the boundary's timestamp), `TurnEndsAfter int` (main-chain turn ends after the boundary in the transcript) and `ReadTurnEndAfter bool` (the turn end the caller passed is the newest of those).

`SessionCycler.CompactedSince` becomes `CompactedSince(turnEnd Event, since time.Time) (CompactionBoundary, bool)`, and `Runner.CompactedSince` returns `(CompactionBoundary, bool, error)`.

`Claude.CompactedSince` in `usage.go` keeps finding the newest main-chain boundary after since, and while walking back from the end to that boundary counts the main-chain turn ends after it: an `assistant` entry, not a sidechain, whose `message.stop_reason` is set and is not `tool_use`.
Claude writes that entry before its Stop hook fires, so a turn end being read is always already in the transcript.
`ReadTurnEndAfter` is true when at least one turn end follows the boundary and the newest one's final text block equals the Stop payload's `last_assistant_message`, or the payload carries none; it is false when both texts are present and differ, which is the watcher reading an older turn end than the transcript's newest.
Every read failure still degrades to not found.

`internal/orchcli/session.go`'s `runnerSession.CompactedSince` adapts the new return to the `orchengine.Session` signature it implements today (`b.At, found, err`) until the next card moves the watcher onto the boundary.
The fake cycler in `session_test.go` takes the new signature.

Tests in `usage_test.go`, over transcript fixtures written inline or under `testdata/`: a boundary followed by no turn end (count 0); by one turn end whose text matches the Stop payload (count 1, read true); by one turn end whose text differs from the payload's (count 1, read false); by two turn ends (count 2); a sidechain assistant entry and a `tool_use` stop reason are not counted; the existing `TestCompactedSince_*` tests assert `At` through the new type.

**ImpactSummary:** The SessionCycler CompactedSince signature changes; its one implementation, the Runner wrapper, the orch adapter and the test fake move with it, and the watcher sees no behaviour change yet.
