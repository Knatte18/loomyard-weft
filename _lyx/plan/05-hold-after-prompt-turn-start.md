# Card 5 — hold-after-prompt-turn-start

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestRun_Wait_HeldTurnEnd`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`
- `internal/shuttleengine#TestWait_LogsSessionStateBesideItsClassification`

**Create:**
- `internal/shuttleengine#Run.armHoldGuard`
- `internal/shuttleengine#turnStartOffsets`
- `internal/shuttleengine#withRunOffset`

**Uses:**
- `internal/shuttleengine#SessionSignalParser`
- `internal/shuttleengine#readEventsFrom`
- `internal/shuttleengine#offsetPastEvent`
- `internal/shuttleengine#heldTurnEnd`
- `internal/shuttleengine#RunState`

**Intent:** The first half of D4: a turn end that precedes the start prompt's own turn start is never a held task turn end, so the skill-load reply can never reach the hold path (#485).

Reading turn starts.
`turnStartOffsets` returns, for each turn-start signal the engine's `SessionSignalParser` reads from `data` (a slice of the events file beginning at `startOffset`), the file offset just past the signal's line, in file order, computed from the signal's `Raw` as `offsetPastEvent` computes an event's; an engine without the capability returns nil.
It goes through the provider-neutral `SessionSignal` kinds only, per PATTERN-shuttle-provider-seam.

The guard.
`Run` gains two fields: `holdGuard`, true while a turn end read is not yet a task turn end, and `promptTurnStarted`, true once a turn start at or past `PromptOffset` has been read; both are in-memory, never persisted.
`armHoldGuard`, called by `Wait` on entry beside `clearWait`, reads the events file once: with the capability present and `PromptOffset` positive, a turn start in the bytes before `PromptOffset` (the load turns' own, which proves the provider writes turn starts for typed turns) arms the guard, and a turn start in the bytes from `PromptOffset` up to `run.offset` (a Wait attached past the prompt's turn start) lifts it at once.
An engine without the capability, a zero `PromptOffset`, a file with no turn start before `PromptOffset`, and an unreadable file leave the guard off, so every such run holds its first turn end as today (AC11).
In `pollEventsTick`, right after `ParseEvents` succeeds and `run.offset` advances, before the `len(events) == 0` return: note whether the guard was armed at the start of the batch, then compute the batch's turn-start offsets with `turnStartOffsets` over the same `data` and `startOffset`; any turn start lifts the guard for the rest of the Wait, whatever the batch's events or its last event, so a turn start read alone in its tick, which yields no event, still lifts it.
Then, before the held turn end is returned: only when the guard was armed at the start of the batch and no turn start of the batch precedes the held turn end's own line (a turn start and a turn end read in one batch are told apart by position) is the turn end not a task turn end: log it at Warn as `shuttle: turn end before the prompt's turn start; not held` with its offset and message, and return no outcome and no held turn end, so the loop keeps polling with the bytes consumed.
Otherwise return the held turn end as today: with the guard off at the start of the batch, the batch's turn starts change no hold, so a turn end followed in one batch by the next turn's start is held and notified as today.
The skip adds no negative-verdict return and no `allOutputFilesExist` call, so the audited sets of `completionsignal_enforcement_test.go` are unchanged and PATTERN-completion-signal holds; `allOutputFilesExist` keeps its place.
The shadow in `waitshadow.go` is never read by the decision; a shadow read failure changes nothing.
Bound, stated in the doc: the guard skips only turn ends before the first post-prompt turn start, and a provider that writes a turn start for the load turns and none for the prompt turn skips every turn end of the run with a Warn each, waiting out the deadline with no hold notice (R3).

Docs: the package doc's held-turn-end paragraph states the rule, its bound and the three cases that keep today's behaviour, and its skill-loading sentence on the offset says the offset is the file size before the prompt is sent (card 4).

Tests.
`withRunOffset` in `fixture_test.go` sets the run's read offset, for the attached-past-the-turn-start row.
`TestRun_Wait_HeldTurnEnd` gains a per-row engine choice (the existing rows keep `fakeEngine`) and a per-step expectation of whether the marker is held, and these rows with `sessionFakeEngine` and a `PromptOffset` set past a seeded `START` and `STOP:loaded` pair: a stray `STOP` appended after the prompt offset holds nothing, then the prompt's `START` appended alone in its own step holds nothing, a `STOP:question` appended in a later step holds, and output and `STOP:done` finish (AC10, and the turn start read alone lifts the guard); a `START` and a `STOP:question` appended in one batch hold at once; a run attached with its offset past the prompt's `START` holds its first `STOP` (AC10 rows); a `sessionFakeEngine` whose liveness answers unproven, so the shadow's facts are incomplete, holds exactly as the first row (shadow failure changes no hold); `fakeEngine` with a positive `PromptOffset`, `sessionFakeEngine` with a zero `PromptOffset`, and `sessionFakeEngine` with no `START` before `PromptOffset` each hold the first `STOP` as today (AC11).
`TestWait_HeldTurnEndNotice` gains an engine choice per row and two rows: with `sessionFakeEngine` and the prompt offset set, a stray `STOP` before the prompt's `START` sends no notice and the `STOP` after it sends one, whose `NotifiedOffset` is the later line's (AC10, no notice); with `sessionFakeEngine` and a zero `PromptOffset`, a `STOP:question` followed by a `START` in one step sends the notice, whose `NotifiedOffset` is the `STOP` line's (AC11, a never-armed guard ignores the batch's turn starts).
`TestWait_LogsSessionStateBesideItsClassification`, which captures the log, gains a row asserting the Warn line for the skipped turn end and that the loop's classification stays `running` across it, then reads `held` at the next turn end.
No new top-level test is added, so the package's row in `cmd/lyx/testdata/test-budget.yaml` stays.

**ImpactSummary:** Two fields and one method on `Run`, one new package function, one branch in `pollEventsTick` and one call in `Wait`; the hold notice path, the gate path and every finalize site are untouched, and nothing outside `internal/shuttleengine` changes.
