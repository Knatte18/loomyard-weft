# Card 1 — neutral-transcript-types

**Create:**
- `plan:internal/shuttleengine#MessageUsage` -> `type MessageUsage struct`
- `plan:internal/shuttleengine#EntryRole` -> `type EntryRole string`
- `plan:internal/shuttleengine#EntryUser` -> `const EntryUser EntryRole = "user"`
- `plan:internal/shuttleengine#EntryAssistant` -> `const EntryAssistant EntryRole = "assistant"`
- `plan:internal/shuttleengine#ToolKind` -> `type ToolKind string`
- `plan:internal/shuttleengine#ToolShell` -> `const ToolShell ToolKind = "shell"`
- `plan:internal/shuttleengine#ToolFileRead` -> `const ToolFileRead ToolKind = "file-read"`
- `plan:internal/shuttleengine#ToolOther` -> `const ToolOther ToolKind = "other"`
- `plan:internal/shuttleengine#ToolCall` -> `type ToolCall struct`
- `plan:internal/shuttleengine#ToolResult` -> `type ToolResult struct`
- `plan:internal/shuttleengine#TranscriptEntry` -> `type TranscriptEntry struct`
- `plan:internal/shuttleengine#TranscriptRead` -> `type TranscriptRead struct`
- `plan:internal/shuttleengine#TranscriptRef` -> `type TranscriptRef struct`
- `plan:internal/shuttleengine#LocatedSession` -> `type LocatedSession struct`
- `plan:internal/shuttleengine#SessionRead` -> `type SessionRead struct`
- `plan:internal/shuttleengine#TranscriptReader` -> `type TranscriptReader interface`
- `plan:internal/shuttleengine#MessagePricer` -> `type MessagePricer interface`
- `plan:internal/shuttleengine#RecordReads` -> `type RecordReads struct`

**Edit:**
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#InstructionFileLister`
- `internal/shuttleengine#UsageReader`

**Intent:** Declare the provider-neutral record-reading seam (D1, D2, D3, D4, D6) in a new file `internal/shuttleengine/transcriptread.go`, beside `UsageReader`, which stays as shuttle's per-run reading (D5).
Declarations only; no behavior, so no test.

- `MessageUsage` (D3): `Input`, `CacheWrite`, `CacheWriteOneHour` (the one-hour part of `CacheWrite`), `CacheRead`, `Output`, all `int`.
  An engine whose provider reports no cache writes or no lifetime split leaves those zero.
- `EntryRole` with `EntryUser` and `EntryAssistant`; `ToolKind` with `ToolShell`, `ToolFileRead` and `ToolOther`.
- `ToolCall`: `ID string`, `Kind ToolKind`, `Command string` (a shell call's command), `Path string` (a file read's path).
- `ToolResult`: `CallID string` (the call it answers), `Text string`.
- `TranscriptEntry` (D2), one user or assistant record in recorded order: `At time.Time` (zero when the record carries no parseable time), `Role EntryRole`, `StartsTurn bool`, `EndsTurn bool`, `MessageKey string` (the provider's message id, or a key the engine makes unique per message), `Model string`, `Usage *MessageUsage` (set only on the message's final record, nil otherwise and on a record the engine skipped as a fork's replay of its parent), `UserTexts []string`, `ToolCalls []ToolCall`, `ToolResults []ToolResult`, `PromptPath string` (set when the entry is the engine's own launch line).
- `TranscriptRef`: `ID string`, the transcript's stable name (for Claude its file name, the name webster records as a fork transcript and tokencount reports as a session), and `Ref string`, an opaque handle only the locating engine reads.
- `LocatedSession`: an embedded `TranscriptRef`, `Modified time.Time` (the last-modified time of the session's own record, its forks' left out, D1/D10), `Forks []TranscriptRef`.
- `TranscriptRead`: `ID string`, `Name string` (the latest strand name lyx set on the session, empty when none), `Started time.Time` (the time of the record's first timestamped line of any kind, zero when none; the Merriam and fork start tokencount reports today), `Repeats int` (D2: streamed records collapsed into their message's final record plus, in a fork, every record of a replayed parent message), `Entries []TranscriptEntry`.
- `SessionRead`: `Main TranscriptRead`, `Forks []TranscriptRead` in `LocatedSession.Forks` order.
- `TranscriptReader`, the locator and reader together: `LocateSessions(worktree string) ([]LocatedSession, error)` returns the sessions the engine recorded under the worktree path, an absent provider root or no session being an empty list and no error (D7); `ReadSession(s LocatedSession) (SessionRead, error)` reads a session the same engine located with its forks, and errors when a transcript cannot be read, a vanished one included (D7).
- `MessagePricer` (D4): `PriceMessage(model string, usage MessageUsage) (usd float64, priced bool)`.
- `RecordReads` (D6), one engine's offline read capabilities: `Transcripts TranscriptReader`, `Pricer MessagePricer`, `Instructions InstructionFileLister`, each nil when the engine lacks it.

`doc.go` gains a paragraph after the session-usage one: the record-reading capabilities, that they read finished records with no run and no shuttle config, that the reader keeps every provider rule and its consumer every lyx-format rule, and that `RecordReads` is what `internal/shuttleengines` hands out.

**ImpactSummary:** additive declarations in `shuttleengine` and one doc paragraph; no existing member changes and no caller is affected.

**Commit:** 1: shuttleengine declares the neutral transcript, usage and pricing capabilities
