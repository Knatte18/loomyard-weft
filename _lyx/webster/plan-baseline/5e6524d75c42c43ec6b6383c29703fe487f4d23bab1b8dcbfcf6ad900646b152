# Card 5 — tokencount-reads-through-engines

**Create:**
- `plan:tools/tokencount#ReadSet` -> `type ReadSet map[string]shuttleengine.RecordReads`
- `plan:tools/tokencount#ReadSet.engines` -> `func (reads ReadSet) engines() (readable, unreadable []string)`
- `plan:tools/tokencount#locatedSession` -> `type locatedSession struct`
- `plan:tools/tokencount#locateSessions` -> `func locateSessions(reads ReadSet, worktree string) ([]locatedSession, error)`
- `plan:tools/tokencount#readLocated` -> `func readLocated(reads ReadSet, s locatedSession) (shuttleengine.SessionRead, error)`
- `plan:tools/tokencount#usageOf` -> `func usageOf(u shuttleengine.MessageUsage) Usage`
- `plan:tools/tokencount#fakeTranscripts` -> `type fakeTranscripts struct`
- `plan:tools/tokencount#fakeTranscripts.LocateSessions` -> `func (f fakeTranscripts) LocateSessions(worktree string) ([]shuttleengine.LocatedSession, error)`
- `plan:tools/tokencount#fakeTranscripts.ReadSession` -> `func (f fakeTranscripts) ReadSession(s shuttleengine.LocatedSession) (shuttleengine.SessionRead, error)`
- `plan:tools/tokencount#fakePricer` -> `type fakePricer map[string]float64`
- `plan:tools/tokencount#fakePricer.PriceMessage` -> `func (p fakePricer) PriceMessage(model string, u shuttleengine.MessageUsage) (float64, bool)`
- `plan:tools/tokencount#oneEngine` -> `func oneEngine(worktree string, sessions ...shuttleengine.SessionRead) ReadSet`

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/cost.go`
- `tools/tokencount#Usage`
- `tools/tokencount#CacheCreation`
- `tools/tokencount#RunTally.observeTime` -> `func (run *RunTally) observeTime(e shuttleengine.TranscriptEntry)`
- `tools/tokencount#RunTally.recordBase` -> `func (run *RunTally) recordBase(entries []shuttleengine.TranscriptEntry)`
- `tools/tokencount#readMerriamStart` -> `func readMerriamStart(read shuttleengine.TranscriptRead) MerriamStart`
- `tools/tokencount#recentRuns` -> `func recentRuns(reads ReadSet, hub, prime string, finished map[string][]ArchiveTag, trees ArchiveTrees, last int) (counted, skipped []string, err error)`
- `tools/tokencount#transcriptStart` -> `func transcriptStart(read shuttleengine.TranscriptRead) (time.Time, bool)`
- `tools/tokencount#CountRun` -> `func CountRun(reads ReadSet, worktree, slug string, bound time.Time) (RunTally, error)`
- `tools/tokencount#RunTally.countFile` -> `func (run *RunTally) countFile(engine string, pricer shuttleengine.MessagePricer, read shuttleengine.TranscriptRead, role string, seen map[string]bool, fork *ForkTally)`
- `tools/tokencount#messageKey` -> `func messageKey(engine string, e shuttleengine.TranscriptEntry) string`
- `tools/tokencount#ForkTally.observe` -> `func (f *ForkTally) observe(e shuttleengine.TranscriptEntry)`
- `tools/tokencount#CountOrch` -> `func CountOrch(reads ReadSet, prime string, runs []RunTally) (OrchTally, error)`
- `tools/tokencount#OrchTally.readTranscript` -> `func (tally *OrchTally) readTranscript(engine string, pricer shuttleengine.MessagePricer, read shuttleengine.TranscriptRead, seen map[string]bool) (string, []orchMessage)`
- `tools/tokencount#assistant` -> `func assistant(id, model string, out, read int) shuttleengine.TranscriptEntry`
- `tools/tokencount#toolResult` -> `func toolResult(ts, text string) shuttleengine.TranscriptEntry`
- `tools/tokencount#toolResultFor` -> `func toolResultFor(useID, ts, text string) shuttleengine.TranscriptEntry`
- `tools/tokencount#bashUse` -> `func bashUse(id, command string) shuttleengine.TranscriptEntry`
- `tools/tokencount#stamped` -> `func stamped(ts string, endsTurn bool) shuttleengine.TranscriptEntry`
- `tools/tokencount#userAt` -> `func userAt(ts, text string) shuttleengine.TranscriptEntry`
- `tools/tokencount#timedAssistant` -> `func timedAssistant(id, model, ts string, output int) shuttleengine.TranscriptEntry`
- `tools/tokencount#userPrompt` -> `func userPrompt(ts, prompt string) shuttleengine.TranscriptEntry`
- `tools/tokencount#TestRunTimeByRole`
- `tools/tokencount#TestCountRun`
- `tools/tokencount#TestCountRunWithoutSessions`
- `tools/tokencount#TestRecentRuns`
- `tools/tokencount#TestShedRecordSpanAndProducers`
- `tools/tokencount#TestCountOrch`
- `tools/tokencount#TestEstimatedCost`
- `tools/tokencount#TestRunRefuses`
- `tools/tokencount#TestRunReadsArchivedShedStatus`
- `tools/tokencount#TestCalibrateGitBackedMatchesInMemory`

**Delete:**
- `tools/tokencount#line`
- `tools/tokencount#lineTime`
- `tools/tokencount#endsTurn`
- `tools/tokencount#startsTurn`
- `tools/tokencount#toolResultItem`
- `tools/tokencount#toolResultItems`
- `tools/tokencount#toolResultTexts`
- `tools/tokencount#bashCommands`
- `tools/tokencount#launchPointerLine`
- `tools/tokencount#userTexts`
- `tools/tokencount#readToolUseIDs`
- `tools/tokencount#nonAlnum`
- `tools/tokencount#projectDir`
- `tools/tokencount#latestTitle`
- `tools/tokencount#isUsageLine`
- `tools/tokencount#lastLinePerMessage`
- `tools/tokencount#eachLine`
- `tools/tokencount#writeLines`
- `tools/tokencount#jsonString`

**Uses:**
- `tools/tokencount#roleOf`
- `tools/tokencount#cardPointers`
- `tools/tokencount#beginBatchCommand`
- `tools/tokencount#workingIntervals`
- `tools/tokencount#fullLoomRun`
- `tools/tokencount#finishedRuns`
- `tools/tokencount#fakeArchiveTrees`
- `internal/shuttleengines#RecordReaders`
- `internal/shuttleengine#RecordReads`
- `internal/shuttleengine#TranscriptReader`
- `internal/shuttleengine#MessagePricer`
- `internal/shuttleengine#TranscriptEntry`
- `internal/shuttleengine#TranscriptRead`
- `internal/shuttleengine#SessionRead`
- `internal/shuttleengine#LocatedSession`
- `internal/shuttleengine#MessageUsage`
- `internal/testkit/shuttlefake#TokencountRunReads`

**Intent:** tokencount reads every session, fork and orch transcript through the engine that recorded it (D1, D2), prices through that engine (D4), and deletes its Claude parser, its projects-root resolution, its `-projects` flag and its price table.
Of the engine packages it imports only `shuttleengine` and `shuttleengines` (`PATTERN-shuttle-provider-seam`), and every lyx-format rule stays here: role from strand name, card pointer lines, the begin-batch envelope, the orch's run attribution, the time-bound cutoff and cross-transcript dedupe.

The read set:
- `ReadSet` is `shuttleengines.RecordReaders()`'s map, taken through a seam: `run` becomes `run(args []string, stdout io.Writer, reads ReadSet) error`, `main` passes `shuttleengines.RecordReaders()`, and tests pass a test-local fake.
- `ReadSet.engines` returns, sorted, the names whose `Transcripts` is set and the names whose `Transcripts` is nil.
- `locateSessions` asks each readable engine in name order for the worktree's sessions and returns them as `locatedSession{engine string; session shuttleengine.LocatedSession}`, sorted by engine then session `ID`; an engine's locator error fails it as `locating sessions of <worktree> through engine <name>: <err>` (D7).
- `readLocated` reads a located session through its own engine and fails any error as `reading session <ID> of engine <name>: %w`, so an unreadable or vanished located session fails the count naming it (D7), and a vanished one's `fs.ErrNotExist` survives the wrap.
- `usageOf` converts `MessageUsage` into tokencount's `Usage`; `Usage` and `CacheCreation` keep their fields and lose their JSON tags, which name Claude's usage fields.

Counting a run (`CountRun`, today's rules over neutral entries):
- It locates `worktree` (`<hub>/<slug>`, which `run` passes) and refuses a run with no located session from any engine as `no sessions for <slug> at <worktree> from engines <readable names>` (D7); main sessions are counted in located order.
- With a non-zero bound, `transcriptStart` (now the earliest `At` of the main read's user or assistant entries, false when none carries one) leaves a session out with its forks and counts it in `LeftOut`, exactly as today.
- The role is `roleOf(read.Main.Name)`; `countFile` tallies one read: every entry with a non-zero `At` widens the run window (`observeTime`) and makes a turn point from its `EndsTurn` and `StartsTurn`; `Duplicates` gains the read's `Repeats`; each entry carrying `Usage` is one message, deduped across the run's transcripts by `messageKey`, the engine name and the entry's message key, so a key repeated across transcripts of one engine counts once and adds one to `Duplicates` (D9), while the same key under two engines counts twice; the message's cost is `messageCost` through the engine's pricer.
- A webster-role session runs `recordBase` over its main entries (a `ToolShell` call whose command matches `beginBatchCommand`, then a tool result answering it whose text is a JSON object with a non-empty `start_sha`, the earliest by the result entry's `At`; a result entry with a zero `At` is skipped, as an unparseable timestamp is today) and `readMerriamStart` over its main read (`Session` the read's `ID`, `Started` the read's `Started`; the prompt path from the first user entry with a `PromptPath`; the first `ToolFileRead` call of that path in a later assistant entry; its result; then the first later assistant entry carrying `Usage`), with today's three `NoStart` reasons.
- Each fork read counts under `<role>+sub`; under a webster session it is a `ForkTally` with `File` the fork read's `ID`, `Session` the main read's `ID` and `Started` the fork read's `Started`, and `ForkTally.observe` takes its cards from the first user entry whose tool-result text holds card pointer lines.

Cost (`cost.go`, rewritten whole): `modelPrice` and `prices` are deleted (card 3 holds them in `claudeengine`), and the file comment says cost.go tallies estimated cost.
`messageCost` becomes `func messageCost(pricer shuttleengine.MessagePricer, model string, u shuttleengine.MessageUsage) Cost`: zero usage costs nothing; a nil pricer, or one answering unpriced, makes a message with nonzero usage unpriced, never free; otherwise the pricer's USD.
`Cost`, its `add`, `share` and `String` are unchanged.

The orch (`CountOrch`): it locates the prime worktree `wd` through every readable engine, counts each session whose role is orch, reads its main entries and its forks through `readTranscript` (the first user entry's joined `UserTexts` as the prompt, each entry carrying `Usage` not yet seen under `messageKey`, kept when its `At` lies within the runs' span, priced through the session's engine), and charges by time or by a single named slug as today; a locator or read error fails it as above.

The default selection (`recentRuns`, D10): its candidates are the slugs of `finished` other than `prime`, each located at `<hub>/<slug>` through every engine; a candidate's recency is the newest `Modified` among its located sessions, and one with no located session is skipped; a locator error fails the selection.
`run`'s no-runs error becomes `no full loom runs of <hub> from engines <readable names>; name the slugs to count a partial run`.

`main.go`, edited whole: the `-projects` flag and the `os.UserHomeDir` projects resolution are gone; `run` passes `filepath.Join(*hub, slug)` to `CountRun`, `wd` to `CountOrch` and `reads` to both and to `recentRuns`.
The `-no-cutoff` help text reads "turn off the archive-tag cutoff, so every located session of each slug counts" and its header line reads "The archive-tag cutoff is off: every located session of each slug counts.".
When `ReadSet.engines` names an unreadable engine, the header gains, after the sleep line and before the skipped line, `Not counted, no transcript reader: <names>.` (D7), printed only then, so an all-Claude report's header is unchanged.
The package comment keeps every report rule and drops every Claude layout and format fact: it says sessions are found per worktree by every registered engine that can locate them and read through the engine that found them, a session's name is the strand name its engine read, a message's usage counts once per engine and message key with the engine collapsing a streamed message, prices come from each engine's pricing capability, a message from an engine without one is unpriced, absence contributes nothing while a locator or read failure fails the count, and the D7 header line; `-projects` and `~/.claude/projects` leave it.

Tests, all over `fakeTranscripts` (sessions keyed by worktree, `SessionRead`s keyed by session `Ref`, an optional locator error, read errors keyed by `Ref`) and `fakePricer` (USD per million tokens of every kind, keyed by model, unknown model unpriced), never writing a provider's transcript format (D6); `oneEngine` builds a one-engine read set named `fake` from main reads, each read's `ID` its `Ref`.
The helpers returning lines now return `TranscriptEntry` values with the neutral fields set directly (`stamped` takes whether the entry ends its turn, `userAt` starts one); `writeLines` and `jsonString` are deleted with their last callers.
Test models are neutral names (`model-a`, `model-b`) priced by `fakePricer` at round fake rates, so no list price or Claude model id stays in tokencount's tests; `orch_test.go`'s file comment and the Opus price comment in `TestCountOrch` say the input is fake reads and the prices are the fake's, and `time_integration_test.go`'s file comment says `-no-cutoff` counts every located session.
- `TestRunTimeByRole` keeps its working-interval rows over entries whose turn flags are set; the "interrupt marker under the cap" and "compact summary after its command" rows are dropped, since with the flags set directly they reach what the tool-result rows already reach, and card 2's `TestReadSession` pins how Claude sets those flags.
- `TestCountRun` takes its main fixture from `shuttlefake.TokencountRunReads()`, card 2's golden of the Claude reader's output, served by `fakeTranscripts`, and keeps today's figures over it: role usage, `Duplicates` of 3 (the reads' `Repeats`), `BaseSHA`, forks with cards, start and peak context; this is the tokencount half of the fixture-reproduction criterion.
  Its other runs are hand-built neutral reads and keep their figures: Merriam starts with their three `NoStart` reasons, the bound leaving out an early session with its fork and a session with no timestamped entry, and a begin-batch result entry with a zero `At` that does not set `BaseSHA`.
  It adds: a key repeated across two sessions of one engine counting once and adding one to `Duplicates`; a second engine on the same worktree whose session is counted beside the first's, with the same message key under each engine counted twice; an engine with no session there beside one with sessions counting the latter.
- `TestCountRunWithoutSessions` becomes a table of `CountRun` and `CountOrch` failures, each asserting the error names its subject: no session from any engine names the engines asked; a locator error names the engine and worktree, through `CountRun` and through `CountOrch`; a read error names the session and engine, through both; a read answering `fs.ErrNotExist` (a session vanished after locating) names the session and satisfies `errors.Is(err, fs.ErrNotExist)`.
- `TestRecentRuns` sets each fake session's `Modified` in place of file times, keeps today's counted and skipped slugs, adds a finished slug with no located session that is skipped, and a row whose locator errors fails the selection.
- `TestShedRecordSpanAndProducers` builds its counted run through `oneEngine`.
- `TestCountOrch` reads the prime's sessions through the fake and keeps every charge and rendered row, with its cost figures recomputed at the fake's rate.
- `TestEstimatedCost` loses its pricing rows to card 3's `TestPriceMessage` and keeps the report rows; it adds a `CountRun` over a one-engine set whose `Pricer` is nil, whose rendered cost cell is `unpriced` (an engine without pricing makes a nonzero message unpriced), and keeps zero usage costing nothing.
- `TestRunRefuses` passes an empty `ReadSet` and gains a row: the retired `-projects` flag is refused as an undefined flag.
- `TestRunReadsArchivedShedStatus` passes each `run` call its own read set.
  The named-slugs call's set holds the fake engine plus an engine `blind` with no `Transcripts`, and it asserts the header line naming `blind`; the `-no-cutoff` call's set holds the fake engine alone and asserts the header lacks any `Not counted, no transcript reader:` line, beside the line's new `-no-cutoff` text; the no-runs call asserts the error naming `fake`; the default call's set holds `done`, `live`, `full` and `solo` with their `Modified` times and no session of `early`, which is added only for the `-no-cutoff` call, so the default header stays as today; the rest is unchanged.
- `TestCalibrateGitBackedMatchesInMemory` drives `run` with its webster session and fork as fake reads.

No new top-level test is added, so the package stays within its budget row.

**ImpactSummary:** every re-signed or deleted member is package-private to `tools/tokencount` and its callers are in this card, the two re-signed through their file targets included: `messageCost` becomes `func messageCost(pricer shuttleengine.MessagePricer, model string, u shuttleengine.MessageUsage) Cost`, called by `countFile`, `readTranscript` and `TestEstimatedCost`, and `run` becomes `func run(args []string, stdout io.Writer, reads ReadSet) error`, called by `main`, `TestRunRefuses`, `TestRunReadsArchivedShedStatus` and `TestCalibrateGitBackedMatchesInMemory`; nothing outside the package imports either.

**Verify:** ! grep -rnE 'input_tokens|cache_creation|cache_read|customTitle|tool_use_id|tool_result|"tool_use"|isSidechain|isCompactSummary|stop_reason|uuid|\.claude|ephemeral_1h|claude-(opus|sonnet|haiku|fable)' tools/tokencount

**Commit:** 5: tokencount counts sessions through each engine's locator, reader and pricing
