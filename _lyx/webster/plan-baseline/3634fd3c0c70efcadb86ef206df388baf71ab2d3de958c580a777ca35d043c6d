# Card 2 — claude-transcript-reader

**Create:**
- `internal/shuttleengine/claudeengine#claudeRecord`
- `internal/shuttleengine/claudeengine#parsedTranscript`
- `internal/shuttleengine/claudeengine#claudeRecord.endsTurn`
- `internal/shuttleengine/claudeengine#claudeRecord.startsTurn`
- `internal/shuttleengine/claudeengine#recordUserTexts`
- `internal/shuttleengine/claudeengine#recordToolCalls`
- `internal/shuttleengine/claudeengine#recordToolResults`
- `internal/shuttleengine/claudeengine#parseTranscript`
- `internal/shuttleengine/claudeengine#launchLine`
- `internal/shuttleengine/claudeengine#projectsRoot`
- `internal/shuttleengine/claudeengine#projectDirUnder`
- `internal/shuttleengine/claudeengine#locateSessions`
- `internal/shuttleengine/claudeengine#Claude.LocateSessions`
- `internal/shuttleengine/claudeengine#Claude.ReadSession`
- `internal/shuttleengine/claudeengine#TestReadSession`
- `internal/shuttleengine/claudeengine#TestLocateSessions`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/a.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/a-webster.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/a-webster/subagents/agent-3.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/b.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/b/subagents/agent-1.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/b/subagents/agent-2.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/c.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/d.jsonl`
- `internal/shuttleengine/claudeengine/testdata/tokencount-run/e.jsonl`
- `internal/testkit/shuttlefake/tokencount-run-reads.json`
- `internal/testkit/shuttlefake#TokencountRunReads`

**Edit:**
- `internal/shuttleengine/claudeengine/usage.go`
- `internal/shuttleengine/claudeengine#claudeProjectDirFor`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/claudeengine#TestSessionUsage_SumsParentAndForks`
- `internal/testkit/shuttlefake/shuttlefake.go`

**Uses:**
- `internal/shuttleengine/claudeengine#claudeHomeDir`
- `internal/shuttleengine/claudeengine#launchPointer`
- `internal/shuttleengine/claudeengine#writeParentTranscript`
- `internal/shuttleengine/claudeengine#writeForkTranscript`
- `internal/shuttleengine/claudeengine#encodeForTest`
- `internal/shuttleengine/claudeengine#assistantLine`
- `internal/shuttleengine/claudeengine#New`
- `internal/shuttleengine#TranscriptReader`
- `internal/shuttleengine#SessionRead`
- `internal/shuttleengine#LocatedSession`
- `internal/shuttleengine#TranscriptEntry`
- `internal/shuttleengine#MessageUsage`

**Intent:** `claudeengine` implements `shuttleengine.TranscriptReader` (D1, D2), and its `SessionUsage` and the reader share one parse of Claude's transcript lines, so the duplicate usage parse ends (D5).
Every Claude transcript fact tokencount's parser holds today moves here, and none stays in tokencount.

The one parse, in `usage.go`: `transcriptTokens` and `tokenSum` give way to `parseTranscript(path, transcriptKey string, includeSidechain bool, skipIDs map[string]struct{}) (parsedTranscript, error)`, declared in `usage.go` beside `SessionUsage`, reading the file line by line with a `bufio.Reader` as `transcriptTokens` does, so a long line is read whole.
It decodes each line into `claudeRecord`: `type`, `uuid`, `timestamp`, `isSidechain`, `isCompactSummary`, `customTitle`, and `message` with `id`, `model`, `stop_reason`, `content` and `usage` (`input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`, `cache_creation.ephemeral_1h_input_tokens`).
A line that does not decode, a half-written last line included, is skipped; an open or read error is returned (an absent file wraps `fs.ErrNotExist`).
`parsedTranscript` holds the transcript's entries, its name (the latest non-empty `customTitle` of a `custom-title` line), its start (the first line's timestamp that parses, whatever the line's type), its repeat count and the set of message ids it counted.
An entry is built for every `user` and `assistant` line whose `isSidechain` is false or `includeSidechain` is set:
- `At` from the RFC 3339 timestamp, zero when it does not parse; `Role`; `EndsTurn` by `claudeRecord.endsTurn` (an assistant line with a stop reason other than `tool_use`, or a user line one of whose texts starts with `[Request interrupted by user` or `<local-command-stdout>`); `StartsTurn` by `claudeRecord.startsTurn` (a user line holding text and no tool result, neither a turn end nor a compaction summary): today's `endsTurn` and `startsTurn` in `tools/tokencount/count.go`, unchanged.
- `MessageKey`: the message id, else the line's uuid, else a key unique to the line across the engine's transcripts (`line:<transcriptKey>:<n>`, `<n>` the record's 1-based line number in its transcript file), so two id-less messages never merge, in one transcript or across transcripts; `Model`.
  `transcriptKey` names the transcript among every transcript the engine records: a session's own ID for its main transcript, and `<session ID>/<fork ID>` for a fork, since a fork's file name is unique only within its session's `subagents/`.
- `UserTexts` by `recordUserTexts` (string content, or the text items of list content); `ToolResults` by `recordToolResults` (each `tool_result` item's `tool_use_id` and its content, a string or text items joined by newlines); `ToolCalls` by `recordToolCalls` (each `tool_use` item: `Bash` is `ToolShell` with `input.command`, `Read` is `ToolFileRead` with `input.file_path`, every other name `ToolOther`).
- `PromptPath`: the path `launchLine` captures from a user text, the multiline pattern of the line `launchPointer` renders, `^Read (.+) in full first; it is your complete, authoritative instructions\.\s*$`, moved here from tokencount's `launchPointerLine`.
- `Usage`: an assistant line's usage converted to `shuttleengine.MessageUsage`, kept only on the message key's last line (Claude's last-line-wins rule for a streamed message, whose earlier lines carry a partial `output_tokens`); each earlier line of the key adds one to the repeat count and carries no usage.
  An assistant line whose id is in `skipIDs` (a fork replaying its parent's spawning message) carries no usage and adds one to the repeat count for each of its lines, and its entry stays for the turn times.

`usage.go`'s file comment drops "All payload and transcript shape knowledge stays in this file" and says the transcript shape knowledge stays in this package, the parse here and the locator in `transcriptread.go`.

`SessionUsage` keeps its observable behavior (D5): the parent read with `includeSidechain` false and no skip set, each fork with its `transcriptKey` as above and `includeSidechain` true and the parent's counted ids as skip set; fresh is input plus cache write plus output and cache read separate, summed over the entries carrying usage; an unreadable parent or fork makes the reading unknown, a missing `subagents/` directory is zero forks, and a directory entry under `subagents/` is skipped, as today.

The locator and reader, in a new file `transcriptread.go`, with `var _ shuttleengine.TranscriptReader = (*Claude)(nil)`:
- `projectsRoot` is `<claudeHomeDir>/.claude/projects`; `projectDirUnder` encodes the workdir under a given projects root, and `claudeProjectDirFor` becomes `projectsRoot` plus `projectDirUnder`, its encoding unchanged.
- `locateSessions(projects, worktree)`: every `*.jsonl` file directly in the encoded project directory is a session, `ID` its file name, `Ref` its absolute path, `Modified` its modification time; its forks are the `*.jsonl` files in `<stem>/subagents/`, sorted by name, a missing `subagents/` directory being zero forks.
  Sessions are sorted by `ID`.
  A missing projects root or project directory is no sessions and no error (D7); any other read error is returned wrapped with the directory.
- `LocateSessions` is `locateSessions` over `projectsRoot()` (D6: the root resolves from `HOME` as the engine already does), so tests run parallel on a temporary root and one row covers the `HOME` default.
- `ReadSession` parses the session's `Ref` with sidechain lines included (today's tokencount counts them), then each fork with the session's counted ids as skip set, each with its `transcriptKey` as above; `TranscriptRead.ID` is the ref's `ID`; any parse error is returned naming the transcript's path, so a transcript that vanished after locating fails (D7).

Tests (`PATTERN-test-speed`: parallel, temporary root; `PATTERN-test-economy`: each a table):
- `TestReadSession`, over transcripts written to a temporary directory with the helpers in `Uses`, one row per scenario of the acceptance criteria: a message streamed over three lines yields its usage once, on its last line's entry, from that line, and counts two repeats; two id-less messages whose lines carry uuids both carry usage, keyed by their uuids; a main transcript and a fork transcript that each hold an id-less, uuid-less assistant usage line at the same line number `n` yield entries whose `MessageKey` values are `line:<main ID>:<n>` and `line:<main ID>/<fork ID>:<n>`, so the two differ; a fork that replays the parent's message id on two lines carries no usage for it and counts both lines in the fork's `Repeats`; the latest of two custom titles names the session, and a later `custom-title` line with an empty `customTitle` leaves it; an interrupt-marker line and a `<local-command-stdout>` line end the turn, a compaction summary line does not start one, a user text line starts one and a tool-result line does not, an `end_turn` assistant line ends one and a `tool_use` one and one with no stop reason do not; a line with no timestamp and one whose timestamp does not parse yield entries with a zero `At`; the line `launchPointer("/p/prompt.md")` renders yields `PromptPath` `/p/prompt.md` as string content and as a text item; a malformed line and a half-written last line are skipped; a `Bash` and a `Read` call and another tool are classed with their command and path, and tool results of string and list content carry their call id and text; a cache write with a one-hour part carries both figures; a sidechain line of the main transcript yields an entry with its usage, since the main read includes sidechain lines; `Started` is the first timestamped line's time; a located session whose main file is removed before the read errors wrapping `fs.ErrNotExist`, and one whose fork file is removed errors naming the fork's path.
- The fixture-reproduction criterion: the transcripts of today's `TestCountRun` main fixture (sessions `a`, `a-webster`, `b`, `c`, `d`, `e` and the forks under `b/subagents/` and `a-webster/subagents/`) are committed under `internal/shuttleengine/claudeengine/testdata/tokencount-run/`.
  Their golden, the `[]shuttleengine.SessionRead` the reader yields for them in located order, encoded with `encoding/json`'s default field names so it holds neutral fields alone, is shared by two packages' tests, so it lives in the `shuttlefake` kit (`PATTERN-testkit`): the kit embeds `tokencount-run-reads.json` and `TokencountRunReads` decodes it into the reader output the golden row compares against, failing loud on a decode error since the file is the kit's own.
  `TokencountRunReads` is declared in `shuttlefake.go`, whose package comment adds that the kit also holds this golden of a Claude reading in neutral types.
  A `TestReadSession` row locates and reads the transcripts directory, with the directory as its project directory, and compares the result with `shuttlefake.TokencountRunReads()` by `reflect.DeepEqual`; the implementer writes the JSON from the reader's own output once the scenario rows pass, and card 5's `TestCountRun` feeds the same golden through its fake, so the two halves cannot drift.
- `TestLocateSessions`, not parallel at the top because one row sets `HOME`: `LocateSessions` under a `HOME` holding one session finds it; over `locateSessions` on a temporary root, in parallel subtests: a missing projects root is no sessions and no error, a missing project directory likewise, two sessions come back sorted with their file times and their forks sorted, a session without `subagents/` has zero forks, a non-`.jsonl` file is ignored, a project directory that is a file errors naming it, and a session whose `subagents` path is a file errors naming it.
- `TestSessionUsage_SumsParentAndForks` gains rows beside the existing ones, which keep their expected readings: two id-less parent lines without uuids both count, pinning D5's "id-less lines included"; an id-less fork line without a uuid counts too, the fork-side counting case.
  These rows pin only that each id-less line counts: `SessionUsage` returns sums and parses each transcript with its own per-message map, so they cannot observe the key, which `TestReadSession` pins; a directory named like a fork transcript under `subagents/` is not a fork, and the reading stays known with that fork left out; a `subagents` path that is a file makes the reading unknown.

The package budget row has room for the two new top-level tests and card 3's one.
`doc.go`'s session-usage paragraph says the reading shares the transcript parse with the reader, and a new paragraph describes the locator and reader: the project directory layout, last-line-wins, the fork replay skip and its repeat count, the turn-flag markers, the launch line and the absence rules.

**ImpactSummary:** `transcriptTokens` and `tokenSum` have `SessionUsage` as their only caller, in the same file; `claudeProjectDirFor` keeps its signature and output for its four callers; the new methods only add capabilities.

**Commit:** 2: claudeengine reads and locates Claude transcripts through one shared parse
