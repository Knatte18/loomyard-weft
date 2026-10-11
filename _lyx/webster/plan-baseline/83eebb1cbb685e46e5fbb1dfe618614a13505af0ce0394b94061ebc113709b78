# Card 2 — Provider-neutral session-start sources

**Create:**
- `plan:internal/shuttleengine#SessionStartSource` -> `type SessionStartSource string`
- `plan:internal/shuttleengine#SessionStartStartup` -> `const SessionStartStartup SessionStartSource = "startup"`
- `plan:internal/shuttleengine#SessionStartResume` -> `const SessionStartResume SessionStartSource = "resume"`
- `plan:internal/shuttleengine#SessionStartClear` -> `const SessionStartClear SessionStartSource = "clear"`
- `plan:internal/shuttleengine#SessionStartCompact` -> `const SessionStartCompact SessionStartSource = "compact"`
- `plan:internal/shuttleengine/claudeengine#sessionStartSource` -> `func sessionStartSource(source string) shuttleengine.SessionStartSource`

**Edit:**
- `internal/shuttleengine#SessionSignal`
- `internal/shuttleengine/claudeengine#signalFromPayload`
- `internal/shuttleengine/claudeengine/signals_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine/testdata/corpus/hook-capture/events.jsonl`

**Intent:**
The provider-neutral half of D3, so `shuttleengine` keys on no Claude string (`PATTERN-shuttle-provider-seam`).
`SessionStartSource` is the source of a session start as `shuttleengine` sees it, with the four constants above; the empty source stands for a missing or unrecognised one.
`SessionSignal.Source` changes type from `string` to `SessionStartSource`, and its doc names the type.
`claudeengine`'s new `sessionStartSource` maps Claude's `startup`, `resume`, `clear` and `compact` to the four constants and any other or missing string to the empty source;
`signalFromPayload` sets `Source` through it.
No fold or readiness rule reads the source yet, so no reading changes.
`claudeengine/doc.go`'s signal paragraph states the mapping.

Tests:
`TestParseSessionSignals` gains rows over `testdata/corpus/hook-capture/events.jsonl` expecting each mapped source on its `SessionStart` line, and a row whose unknown source string reads the empty source.

**ImpactSummary:** `SessionSignal.Source`'s only writer is `signalFromPayload`, and no production code reads it yet; the existing test literals stay untyped constants and compile unchanged.
