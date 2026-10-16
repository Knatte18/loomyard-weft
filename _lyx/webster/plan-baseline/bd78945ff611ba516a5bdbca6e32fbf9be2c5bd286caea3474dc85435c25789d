# Card 3 — shuttle-box-reads

**Create:**
- `plan:internal/shuttleengine#boxCapture` -> `func boxCapture(reed ReedOps, engine Engine, guid string) (string, error)`
- `plan:internal/shuttleengine#blankMarkedSpans` -> `func blankMarkedSpans(styled string) string`
- `plan:internal/shuttleengine#styledReadinessEngine` -> `type styledReadinessEngine struct`
- `plan:internal/shuttleengine#styledReadinessEngine.PlainBoxCapture` -> `func (e *styledReadinessEngine) PlainBoxCapture(styled string) string`
- `plan:internal/shuttleengine#styledPasteEngine` -> `type styledPasteEngine struct`
- `plan:internal/shuttleengine#styledPasteEngine.PlainBoxCapture` -> `func (e *styledPasteEngine) PlainBoxCapture(styled string) string`
- `plan:internal/shuttleengine#tuiReed.CapturePaneStyled` -> `func (r *tuiReed) CapturePaneStyled(guid string) (string, error)`

**Edit:**
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#busyReading`
- `internal/shuttleengine#readInputBox`
- `internal/shuttleengine#IdleProbe`
- `internal/shuttleengine#tuiReed`
- `internal/shuttleengine#tuiReed.CapturePane`
- `internal/shuttleengine#TestRunner_SessionIdle_ReturnsScriptedClassification`
- `internal/shuttleengine#TestRun_Send_ConfirmsSubmission`
- `internal/shuttleengine/doc.go`
- `//pattern/PATTERN-pane-text-injection.md`
- `//internal/orchengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine#StyledBoxReader`
- `internal/shuttleengine#ReedOps`
- `internal/shuttleengine#readBox`
- `internal/shuttleengine#readinessEngine`
- `internal/shuttleengine#strayPasteEngine`
- `internal/shuttleengine#pasteEngine`
- `internal/shuttleengine#newTUIEngine`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#liveStrandStatus`

**Intent:** Decision D1, shuttle half, and the shuttle part of D2: every box reading of a send or idle probe reads one capture, styled where the engine can blank its placeholder, and the pane-only probe reports a draft.

`boxCapture` is the one place the plain-versus-blanked choice is made: for an engine with `StyledBoxReader` it takes `reed.CapturePaneStyled(guid)` and returns `PlainBoxCapture` of it, and for any other engine it returns `reed.CapturePane(guid)`; an error is the capture's, unchanged.
Its callers are exactly the box readings D1 names, each taking one capture per decision so the running-turn, readiness and box readings of that decision come from one screen:
`Runner.SessionIdle`'s capture, on the readiness path and the pane-only fallback alike, which is the idle probe with and without a run record;
`busyReading`'s capture, which is the pre-write guard of every verified send, its stray-character clear and its post-clear reading included;
and `readInputBox`, which is the landing check `confirmPasted`, `awaitSettledBox` and `clearUnlandedText` read the box through.
Every other capture (`requireReadyAgentPane`, `guardedInterrupt`, `closePickerBeforeSend`, `closeRewindPicker`, `conditionedReading`, `paneTail`, the needle scans of `typeVerified`, the wait loop's) keeps `CapturePane`, as D1 says.
An engine without the capability reads exactly as today, so only Claude's readings widen.

`Runner.SessionIdle`'s pane-only fallback, taken for an unknown readiness reading and for a strand with no run record, now fills `IdleProbe.Draft` too: an answer that is not idle sets `Draft` when `readBox` of the same capture reads a box whose trimmed text is non-empty.
`IdleProbe.Draft`'s doc drops "the hold the readiness reading names" for "text in the input box, on either reading".
`TooShort` is filled as today; Claude reads no box in a pane too short to draw one, so its probe never sets both, and creel's mapping prefers the short pane for an engine that does (card 6).
The orch watcher's `heldReason` already reads `Draft` before its pane-not-idle reason, so an orch probe on the fallback now names the draft, the more precise reason, and `internal/orchengine/doc.go`'s residual that the idle probe fails closed on any non-empty box gains "other than the provider's placeholder".

The fakes: `blankMarkedSpans` replaces every `~…~` span of a fake capture, tildes included, by spaces, standing in for a provider's placeholder cells.
`styledReadinessEngine` embeds `readinessEngine` and `styledPasteEngine` embeds `*strayPasteEngine`, each answering `PlainBoxCapture` with `blankMarkedSpans`.
`tuiReed` gains a `suggestion` field: while its box is empty, `tuiReed.CapturePane` draws the suggestion after the caret as plain text, as a plain capture of Claude does, and `tuiReed.CapturePaneStyled` draws it wrapped in `~` marks; an empty `suggestion` draws nothing, so every existing row reads as before.

Rows, so a broken routing fails them:
`TestRunner_SessionIdle_ReturnsScriptedClassification` gains a `styled` row field choosing `styledReadinessEngine`, and rows where a hook-ready reading over `IDLE\n❯ ~Try something~` answers `{Idle: true}` on the styled engine and the draft hold quoting `~Try something~` on the plain one, and where an unknown reading over the same capture falls back to an idle pane on the styled engine;
a fallback row where an unknown reading over `BUSY\n❯ half a draft` answers `{Draft: true}`;
and a subtest over a fixture with no run record (`newFixture` without `withStrand`, the reed reporting the strand live) whose rows answer `{Idle: true}` for an idle pane, `{Draft: true}` for `BUSY\n❯ half a draft`, since `readinessEngine`'s idle reading looks only at the `IDLE` prefix, and `{Idle: true}` for `IDLE\n❯ ~Try something~` on the styled engine;
and a row on the styled engine whose reed fails `CapturePaneStyled` through `CaptureErr` answers an error wrapping it, so `boxCapture`'s error reaches the probe's caller.
`TestRun_Send_ConfirmsSubmission`'s paste table gains `suggestion` and `styled` fields: with the suggestion `Try something` on a `tuiReed`, a send over `styledPasteEngine` pastes once and is confirmed, the box reading empty both at the pre-write guard and after the paste lands;
the same row over the plain `strayPasteEngine` fails with a draft hold wrapping `ErrSessionBusy` that quotes the suggestion and pastes nothing, which is #657's stall.

Docs in the same commit: `internal/shuttleengine/doc.go` names the styled box read beside the pre-write guard paragraph and the pane-only probe's `Draft`, and `pattern/PATTERN-pane-text-injection.md` adds, beside the lone `%`, that Claude's box reading through a styled capture takes a box holding only placeholder-styled text as empty, with D1's bound: only when every non-blank cell, after the caret, side bars and a lone `%` are stripped, is drawn in the placeholder style, so a typed draft, drawn unstyled, still reads as a draft, and a restyled placeholder reads as a draft, fail-closed.

**ImpactSummary:** Three shuttle reads and the fallback probe change; only engines with StyledBoxReader (Claude) read differently, and SessionIdle's callers (creel, the orch watcher) see Draft set on the pane-only path.
