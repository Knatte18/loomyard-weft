# Card 2 — A session's forks count as its activity

**Create:**
- `plan:internal/shuttleengine/claudeengine#forkActivity` -> `func forkActivity(transcriptPath string, now time.Time) time.Time`
- `plan:internal/shuttleengine/claudeengine#holdsPendingToolCall` -> `func holdsPendingToolCall(path string) bool`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude`
- `internal/shuttleengine/claudeengine#New`
- `internal/shuttleengine/claudeengine#NewFromConfig`
- `internal/shuttleengine/claudeengine/activity.go`
- `internal/shuttleengine/claudeengine/activity_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#ActivityReader`
- `internal/shuttleengine#TurnEndActivity`
- `internal/shuttleengine#AgentActivity`
- `internal/shuttleengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine/claudeengine#bashMaxTimeout`
- `internal/shuttleengine/claudeengine#forEachTranscriptLine`
- `internal/shuttleengine/claudeengine#transcriptLine`
- `internal/shuttleengine/claudeengine#transcriptBlock`
- `internal/shuttleengine#ReadAgentActivity`

**Intent:**
Decision record D9: once forks run gates in the foreground they send no interim notices, so a Merriam waiting on a working fork would read quiet for the whole batch, and loom's step idle kill and batten's quiet notice would fire.
The turn-end activity reading therefore takes the newest activity across the main transcript and the session's fork transcripts.
The reading stays inside claudeengine: `ReadAgentActivity` and its consumers change no code.

`Claude` gains an unexported field `now func() time.Time`, the reading clock.
`New` and `NewFromConfig` set it to `time.Now`.
The `Claude` doc comment and `claudeengine.go`'s file header say the methods are pure functions of their arguments, the settle fields and that clock.

`TurnEndActivity` keeps its main-transcript reading (modification time, API-error walk) and its zero reading on a missing path or unreadable main transcript.
It then sets `TranscriptModTime` to the later of the main transcript's modification time and `forkActivity(payload.TranscriptPath, c.now())`.
The API-error part still reads the main transcript only.

`forkActivity` reads the `subagents/` directory beside the transcript: `strings.TrimSuffix(transcriptPath, ".jsonl")` joined with `subagents`, the same layout `SessionUsage` and `AuditForksIncremental` read.
It returns the zero time when the directory is missing or unreadable.
For each regular file ending in `.jsonl` it takes the file's modification time;
when `now` is no later than that time plus `bashMaxTimeout` and `holdsPendingToolCall` reports true, the fork's reading is `now`, otherwise its modification time.
A fork transcript that cannot be stat'ed or read is skipped.
It returns the newest reading across the forks.
A fork last written more than the ceiling ago is never scanned, so a long Master session does not re-read its finished forks' transcripts on each reading.

`holdsPendingToolCall` streams the file through `forEachTranscriptLine` and reports whether some `tool_use` block's `id` has no `tool_result` block naming it in `tool_use_id`, wherever the call sits, so a held gate followed by a parallel sibling call's result still reads pending.
A read error reports false.

Bound (D9): a fork stopped mid-call leaves a pending call with no writer and reads active for at most `bashMaxTimeout` past its last write;
a live call ends or backgrounds at that same ceiling, so the cap never cuts a held call short, and a fork with no call in flight reads only its last write.

Docs:
`TurnEndActivity.TranscriptModTime` and `AgentActivity.LastActivity` in `internal/shuttleengine/activity.go`, and the agent-activity paragraph of `internal/shuttleengine/doc.go`, say the session's last activity, its forks included, rather than the transcript's last write;
the field names stay.
The `ActivityReader.TurnEndActivity` interface method doc, the `ActivityReader` and `TurnEndActivity` type docs, the `Claude.TurnEndActivity` doc and the file header of `claudeengine/activity.go` likewise say the reading is the session's last activity, its forks included, with a fork's pending call reading active up to `bashMaxTimeout` past its last write, and that the API-error part reads the main transcript only;
none keeps a sentence calling the reading the transcript's last write time.
`claudeengine/activity.go` is a whole-file target because its file header changes beside `Claude.TurnEndActivity`.
The activity-reading paragraph of `claudeengine/doc.go` describes the fork part, the pending-call reading and its ceiling cap.

Tests, in the existing `TestTurnEndActivity` (no new top-level test):
its table construction passes a `Claude` with `now` fixed, and a new table of fork rows writes a main transcript and fork transcripts under `<session>/subagents/` with `os.Chtimes`-set times, asserting `TranscriptModTime` exactly:
a fork written after the main transcript reads its write time;
a fork holding a `tool_use` with no result, written within the ceiling, reads `now`;
a pending call followed by a sibling call's `tool_result` still reads `now`;
a pending call last written more than the ceiling before `now` reads its write time;
a fork whose every call has its result, older than the main transcript, leaves the main transcript's time.
The ceiling offset in the rows is built from `bashMaxTimeout`.

**ImpactSummary:** the claude activity reading feeding loom's step idle kill and batten's quiet notice now counts fork transcripts; no consumer changes code, and the `Claude` type gains a clock field set by its constructors.

**Commit:** 2: claudeengine counts a session's forks in its turn-end activity
