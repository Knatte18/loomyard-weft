# Card 2 — claudeengine-turn-progress

**Create:**
- `internal/shuttleengine/claudeengine#Claude.TurnStartProgress`
- `internal/shuttleengine/claudeengine#newestTurnProgress`
- `internal/shuttleengine/claudeengine#scanTurnStartTranscript`
- `internal/shuttleengine/claudeengine#TestTurnStartProgress`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/prompt.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/task-notification.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/tool-result.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/tool-use.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/end-turn.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/interrupt.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/thinking-tool-use.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/thinking-end-turn.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/api-error.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/null-stop-reason.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/max-tokens.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/trailing-skipped.jsonl`
- `internal/shuttleengine/claudeengine/testdata/turnprogress/no-trailing-newline.jsonl`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.TurnStartInterrupt`
- `internal/shuttleengine/claudeengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D2: Claude implements card 1's `shuttleengine.TurnProgressReader` by reading the transcript the turn start's `UserPromptSubmit` payload names, so shuttle tells a running turn from an orphaned turn start.
Transcript entry shapes, stop reasons and the interrupt marker stay in `claudeengine` (`PATTERN-shuttle-provider-seam`).

`prober.go` gains `var _ shuttleengine.TurnProgressReader = (*Claude)(nil)`, and its file comment names the new reading beside liveness and the interrupt marker.

`scanTurnStartTranscript` holds the open-and-walk both readings share: it decodes `transcript_path` from `turnStart.Raw`, opens and stats the file, and runs `readBackward(f, size, initialReadChunk, scan)`; a missing path, an undecodable payload, or an open or stat failure returns without calling `scan`.
`Claude.TurnStartInterrupt` keeps its behaviour and moves onto it, so one transcript-reading path serves both.

`Claude.TurnStartProgress` returns `TurnProgressUnknown` unless `scanTurnStartTranscript` lets `newestTurnProgress` find an entry, in which case it returns that entry's reading.

`newestTurnProgress` first drops every byte after the last newline of `data`, so a line still being written (no trailing newline) is never read; it then scans the remaining complete lines from the last to the first, as `newestEntryInterrupt` does.
A blank line, a line that does not parse as JSON, a sidechain entry (`isSidechain: true`) and any entry whose `type` is neither `user` nor `assistant` (`system`, attachments, summaries, `last-prompt`, `cost-state`, any unknown type) is skipped; unknown keys are ignored (`PATTERN-tolerant-records`).
The first entry it does not skip decides, with `found` true:
- a `user` entry whose final text starts with `interruptMarkerPrefix` reads `TurnProgressUnknown`, since D1 releases on a reported interrupt first;
- any other `user` entry reads `TurnInFlight`: a prompt, a task notification, a tool result;
- an `assistant` entry is classified by its message's `stop_reason`, never by its last content block: `tool_use` reads `TurnInFlight`, `end_turn` and `stop_sequence` (which Claude Code's synthetic API-error entry carries) read `TurnOver`, and an absent, null or any other value (such as `max_tokens`) reads `TurnProgressUnknown`.
Every line of a completed message split over several transcript lines carries the message's final stop reason, so a thinking or text line ahead of its `tool_use` line already reads in flight.
`transcriptEntry.Message.StopReason` decodes a null as `""`, which falls in the unknown branch.

Fixtures are files in `testdata/turnprogress/`, one JSON object per line, each line ending in a newline unless the file says otherwise.
Each fixture line whose shape appears in the bench transcripts (a string prompt, a task notification, a `tool_result` user entry, a `tool_use` assistant line, a thinking line with `stop_reason: "tool_use"`, a thinking line with `stop_reason: "end_turn"`, an `end_turn` text line, a `system` `turn_duration` entry) is copied verbatim from `ftp-r-1` or `ftp-s-1` under the hub prime's `.scratch/twill-fork-timing-plan/after/<run>/transcripts/`, so all of the line's real keys stay.
Only shapes the bench lacks (the interrupt marker, the API-error entry, the null and `max_tokens` stop reasons, the unknown type, the malformed and unterminated lines) are hand-built, each carrying the shape of a Claude Code transcript entry: `type`, `isSidechain`, `timestamp`, `uuid`, and for a message entry `message` with `role`, `content` and, on an assistant entry, `stop_reason`.
Each file opens with an assistant `end_turn` entry closing a previous turn, so a reading that skipped past the newest entry would read over:
- `prompt.jsonl`: then a user entry whose content is a string prompt → `TurnInFlight`.
- `task-notification.jsonl`: the bench incident shape: then a `system` entry with `subtype: "turn_duration"`, then a user entry whose content is a string starting `<task-notification>\n<task-id>` and naming a completed agent → `TurnInFlight`.
- `tool-result.jsonl`: then a user prompt, an assistant `tool_use` entry, and a user entry whose content is one `tool_result` block → `TurnInFlight`.
- `tool-use.jsonl`: then a user prompt and an assistant entry whose content is one `tool_use` block, `stop_reason: "tool_use"` → `TurnInFlight`.
- `end-turn.jsonl`: then a user prompt and an assistant entry with one text block, `stop_reason: "end_turn"` → `TurnOver`.
- `interrupt.jsonl`: then a user prompt and a user entry whose content is a text block `[Request interrupted by user]` → `TurnProgressUnknown`.
- `thinking-tool-use.jsonl`: then a user prompt and an assistant entry with one `thinking` block and `stop_reason: "tool_use"`, with no `tool_use` line after it → `TurnInFlight`.
- `thinking-end-turn.jsonl`: then a user prompt and an assistant entry with one `thinking` block and `stop_reason: "end_turn"` → `TurnOver`.
- `api-error.jsonl`: then a user prompt and an assistant entry with `isApiErrorMessage: true`, one text block starting `API Error:` and `stop_reason: "stop_sequence"` → `TurnOver`.
- `null-stop-reason.jsonl`: then a user prompt and an assistant text entry with `stop_reason: null` → `TurnProgressUnknown`.
- `max-tokens.jsonl`: then a user prompt and an assistant text entry with `stop_reason: "max_tokens"` → `TurnProgressUnknown`.
- `trailing-skipped.jsonl`: then a user prompt, an assistant `end_turn` entry, a sidechain user prompt (`isSidechain: true`), a `system` entry, an entry of an unknown `type`, and a last line `{"type":"user","message":` that does not parse → `TurnOver`.
- `no-trailing-newline.jsonl`: then a user prompt and an assistant `end_turn` entry, each newline-terminated, then a complete user prompt line with no trailing newline → `TurnOver`, since the unterminated line is not read.

`TestTurnStartProgress` in `prober_test.go` is a table over these fixtures: each row builds a `UserPromptSubmit` payload whose `transcript_path` is the fixture's path, calls `(&Claude{}).TurnStartProgress` and asserts the exact `TurnProgress`.
Two more rows assert `TurnProgressUnknown` for a payload without `transcript_path` and for a path to a file that does not exist.
It is a new top-level test because no existing test covers `TurnStartProgress`; `TestTurnStartInterrupt` covers the other method of the shared reader and keeps passing over `scanTurnStartTranscript`.
`test-budget.yaml`'s `internal/shuttleengine/claudeengine` row goes from 110 to 111.

`doc.go`'s paragraph on the prober (the lines after "The prober also reads the interrupt marker.") gains the turn-progress reading: which newest main-chain entry reads in flight, over or unknown, that an assistant entry is read by its message's stop reason, that an unterminated last line is not read, and that every read failure reads unknown.

**ImpactSummary:** one existing method moves onto a shared private reader with unchanged behaviour; one new method on `Claude` implementing an optional shuttleengine capability; no caller outside the package changes.

**Commit:** 2: claudeengine: read a turn start's progress from its transcript
