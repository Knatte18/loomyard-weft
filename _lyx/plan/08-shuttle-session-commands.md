# Card 8 — session commands and the color step through the guarded path

**Create:**
- `plan:internal/shuttleengine#sendSequence` -> `func sendSequence(sc sendContext, inputs []PaneInput) error`
- `plan:internal/shuttleengine#sequenceText` -> `func sequenceText(inputs []PaneInput) string`

**Edit:**
- `internal/shuttleengine#Runner.ClearSession`
- `internal/shuttleengine#Runner.ReloadPlugins`
- `internal/shuttleengine#Runner.CompactSession`
- `internal/shuttleengine#Runner.TypeColor`
- `internal/shuttleengine#typeColor`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#TestRunner_ClearSession_PlaysScriptedSequence`
- `internal/shuttleengine#TestRunner_TypeColor_PlaysTheSequenceAndWaitsItOut`
- `internal/shuttleengine#TestRunner_ReloadPlugins_PlaysSequenceOnALiveShuttleStrand`
- `internal/shuttleengine#TestRunner_CompactSession_PlaysSequenceForFocus`
- `internal/shuttleengine#TestRunner_Start_TypesTheSegmentColorAfterStartup`
- `internal/shuttleengine/doc.go`

**Delete:**
- `internal/shuttleengine#colorSettleAttempts`
- `internal/shuttleengine#colorSettleInterval`

**Uses:**
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#Runner.holdFor`

**Intent:**
Put `/clear`, `/compact`, `/reload-plugins` and `/color` behind the same pre-write guard, paste-submit and confirmation as every other write (decision D6), so none of them types on a caller's earlier probe.

`sequenceText` joins the `Text` of a choreography's steps and ignores its keys and pauses.
`sendSequence(sc, inputs)` sends a choreography as one write: for a paste-path engine (`TextPaster` with `InputBoxReader` and `SessionCycler`) it is `sendVerified(sc, sequenceText(inputs))`, so the slash command is pasted once and confirmed by the box reading empty, since it starts no turn; for any other engine it runs `awaitIdleSession` and then `playInputs`, keeping that engine's own keys behind the same guard.

`Runner.ClearSession`, `Runner.ReloadPlugins` and `Runner.CompactSession` find the run through `FindRun`, build `r.newSendContext(state, runDir)` with `sc.hold = r.holdFor(guid)`, and call `sendSequence` over the cycler's sequence, in place of `requireLiveStrand` plus a bare `playInputs`; their doc comments drop "the caller has already probed idleness itself".
`typeColor(sc, color)` becomes `sendSequence(sc, engine.ColorSequence(color))` and returns its error, held or not; the settle loop and its two constants go.
`Runner.TypeColor` returns that error, so the orch watcher can retry a held color (card 11); `Runner.start`'s color step still only logs a failure, since the color is display only and must not fail a launch.

What this lets through and what bounds it: a session command into a box holding a draft is now refused with an error wrapping `ErrSessionBusy` after the idle wait, where it used to type at once; the caller retries (card 11).

Tests:
- `TestRunner_ClearSession_PlaysScriptedSequence`, `TestRunner_CompactSession_PlaysSequenceForFocus` and `TestRunner_ReloadPlugins_PlaysSequenceOnALiveShuttleStrand` gain rows: with a drafted box the command types nothing and fails wrapping `ErrSessionBusy` even though the caller's own probe was idle; with an active operator it is held; with an empty box a paste-path engine issues one `PasteSubmit` of `/clear` (resp. `/compact <focus>`, `/reload-plugins`) and no `SendText`.
- `TestRunner_TypeColor_PlaysTheSequenceAndWaitsItOut`: a held color returns an error wrapping `ErrSessionBusy`, and a pasted `/color <name>` is confirmed by the empty box.
- `TestRunner_Start_TypesTheSegmentColorAfterStartup` keeps passing with the color pasted, and a held color still starts the run.

Update the session-cycling paragraph of `internal/shuttleengine/doc.go`.

**ImpactSummary:** Four Runner session methods and start's color step change how they type; orchengine is their only caller and absorbs the new held error in card 11.
