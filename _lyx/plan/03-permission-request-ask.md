# Card 3 — PermissionRequest reads asking at once, and every Notification type is mapped

**Create:**
- `plan:internal/shuttleengine/claudeengine#notificationElicitationURLDialog` -> `const notificationElicitationURLDialog = "elicitation_url_dialog"`
- `plan:internal/shuttleengine/claudeengine#notificationSignalKinds` -> `var notificationSignalKinds map[string]shuttleengine.SessionSignalKind`

**Edit:**
- `internal/shuttleengine/claudeengine#signalFromPayload`
- `internal/shuttleengine#SessionFold.applyTranscriptMarkers`
- `internal/shuttleengine/claudeengine/signals_test.go`
- `internal/shuttleengine/sessionstate_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
D5's parsing and fold half; card 1 installed the recording hook.

`signalFromPayload` maps a `PermissionRequest` payload line to `SessionSignalAsk` by the hook name alone, with `Event` false, since `ParseEvents` reads no event from it.
The `Notification` mapping becomes the fixed table `notificationSignalKinds`: `permission_prompt`, `elicitation_dialog` and `elicitation_url_dialog` are asks, `idle_prompt` is the idle notice, and `agent_needs_input`, `elicitation_response` and every other type yield no signal (absent from the table).

`SessionFold.applyTranscriptMarkers` releases an ask with cause `ask` to idle-stalled with cause `interrupt` on the transcript's interrupt marker of the newest turn start, as it releases a running turn, when the marker's time is not before the ask's `Since` (or either time is zero).
An ask from an `AskUserQuestion` call or a `Notification` is released the same way, since all carry cause `ask`; an awaiting-input ask from a turn end is not.
Bound: an ask holds every send until the next turn signal or an interrupt marker; a granted permission writes neither, so it reads asking for the rest of its turn.

Tests:
`TestParseSessionSignals` gains rows over the captured `PermissionRequest` and `Notification` lines in `testdata/hooks/events.jsonl` and one row per table entry, an unknown type yielding no signal;
`TestSessionFold` gains rows: `PermissionRequest` reads asking, an interrupt marker past that ask reads idle-stalled with cause interrupt, and a marker older than the ask leaves it asking.
`claudeengine/doc.go` and `shuttleengine/doc.go`'s fold section state the table and the release.

**ImpactSummary:** a permission dialog reads asking about six seconds sooner and holds sends until a turn signal or an interrupt; an `elicitation_url_dialog` notification now reads as an ask too, and an ask an interrupt marker follows no longer holds sends.
