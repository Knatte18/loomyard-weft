# Card 4 — Typed text settles, submission is confirmed in a window, an unlanded send clears its text

**Create:**
- `plan:internal/shuttleengine#ErrSubmissionNotLanded` -> `var ErrSubmissionNotLanded error`
- `plan:internal/shuttleengine#submitConfirmTimeout` -> `func submitConfirmTimeout(cfg Config) time.Duration`
- `plan:internal/shuttleengine#confirmBackoffCap` -> `const confirmBackoffCap = 5 * time.Second`
- `plan:internal/shuttleengine#awaitSettledBox` -> `func awaitSettledBox(sc sendContext, reader InputBoxReader, closeAt time.Time) error`
- `plan:internal/shuttleengine#clearUnlandedText` -> `func clearUnlandedText(sc sendContext, reader InputBoxReader, normalized, needle string) (landed bool, note string)`
- `plan:internal/shuttleengine#eventsSize` -> `func eventsSize(path string) int64`
- `plan:internal/shuttleengine#turnStartedSince` -> `func turnStartedSince(engine Engine, eventsPath string, offset int64) bool`

**Edit:**
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#confirmSubmitted` -> `func confirmSubmitted(sc sendContext, reader InputBoxReader, normalized, needle string, sentAt int64, closeAt time.Time) error`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/doc.go`

**Delete:**
- `internal/shuttleengine#sendExtraEnters`

**Uses:**
- `internal/shuttleengine#InputBoxReader`
- `plan:internal/shuttleengine#InputBoxClearer`
- `internal/shuttleengine#SessionCycler`
- `internal/shuttleengine#inputBoxHoldsSentText`
- `plan:internal/shuttleengine#paneTail`

**Intent:**
Implement D2, D3, D5's tail for the landing failure, and the "text never appeared" decision: a send that does not land fails with `ErrSubmissionNotLanded` and leaves the box as empty as D1 found it.
The new helpers join `send.go`.

`sendVerified`, after card 3's idle wait:

- reads `eventsSize(sc.eventsPath)` as the pre-send offset, then opens the submit window, `submitConfirmTimeout(sc.cfg)` from the moment typing begins, cut to `sc.deadline` when that is earlier.
- for an engine that implements `InputBoxReader`, types with `TypeSequence`, runs the existing appearance check (count and position, `sendVerifyAttempts` polls `sendVerifyInterval` apart, one replay), then `awaitSettledBox`, then `confirmSubmitted`; an engine without it keeps `ComposeSend`, the appearance check alone and no window, as today.
- sleeps the appearance polls through `sc.clock` and cuts the polls and the replay short when the window closes; `playInputs`' per-step `SettleMS` stays on `inputSleep`.
- reports a text that never appeared (replays spent or window closed) with today's "never appeared" message, now wrapping `ErrSubmissionNotLanded` and ending with `paneTail`.

`awaitSettledBox` reads the input box every `sc.cfg.SubmitSettleMS` until two consecutive reads show the same content (a collapsed paste placeholder is content like any other) and returns nil; a box that has not settled when the window closes returns `ErrSubmissionNotLanded`, and no Enter is sent.

`confirmSubmitted` sends the first Enter, then reads the box at an interval that starts at `reader.SubmitSettle()` (the engine's `submit_redraw_settle_ms`) and doubles up to `confirmBackoffCap`.
An interval that would end past `closeAt` is cut to end at it, but the read after an Enter never comes sooner than `SubmitSettle()` after it.
A read taken while the window is open that shows the box still holding the sent text (`inputBoxHoldsSentText`, unchanged) sends one more Enter; one showing an empty box, a running turn or a draft lacking the needle sends none, as today's guard.
A read taken at or after `closeAt` sends no Enter and ends the loop, so every Enter is followed by exactly one confirm read.
The submission is confirmed when the box no longer holds the sent text, or, for an engine with session signals, when `turnStartedSince(sc.engine, sc.eventsPath, sentAt)` finds a `turn_start` past the pre-send offset, even while the box reading is ambiguous.
`sendExtraEnters` and its fixed count go.

On every `ErrSubmissionNotLanded` from the box path (unsettled, never confirmed, never appeared), `sendVerified` first calls `clearUnlandedText`: it reads the box once; an empty box means the submission landed late, and the send returns nil instead of failing (D4's first catch).
Only for an engine with both the idle reading (`SessionCycler`) and `InputBoxClearer`, and only when that reading shows this send's own text, does it play `ClearInputSequence` and read the box once more.
Its own text means: one or more copies of the normalized sent text, the last of which may be a prefix at least as long as the needle, or a reading `PastePlaceholder` accepts.
A box holding anything else gets no clear key; one still not empty after the clear is named in the returned note.
The error wraps `ErrSubmissionNotLanded`, says why (unsettled, still pending after the window, never appeared), carries that note, and ends with `paneTail`.

`submitConfirmTimeout` floors a non-positive hand-built value to the template default 30 s.

Bound (this card replaces a fixed Enter count with a window and adds a clear key): extra Enters still go only into a box holding the sent text, and the window caps how long and therefore how many.
The clear goes only into a box an idle reading proved empty before typing and that now holds this send's own text, never into a foreign draft or an empty box, and uses no key that interrupts a turn.
The window's only overshoot is the last confirm read, the clear with its two box readings, and `paneTail`'s capture.

Tests: `TestRun_Send_ConfirmsSubmission`'s table is rewritten to the window on a `fakeClock`.
Its `inputBoxEngine` gains a scripted `InputBoxClearer` and, per row, card 3's idle-reading fake from `fakes_test.go` (`SessionCycler` scripted to read idle at once), since the clear runs only for an engine with both; those rows script the idle wait's pane captures ahead of the box answers, and the row for an engine without the idle reading omits it.
The table keeps its existing rows (empty at once, held once then cleared, wrapped long text, foreign draft, longer draft containing a short text, paste placeholder, no readable box) and adds rows for: a box that changes between reads getting no Enter until two reads agree; a box that never settles (`ErrSubmissionNotLanded`, no Enter); Enters at a growing interval until the box clears; an Enter at the window's close followed by one confirm read and no further Enter; a `turn_start` past the pre-send offset confirming; the pre-clear reading empty succeeding; a doubled replay, a needle-long prefix and a placeholder each cleared to empty; a foreign draft and an empty box getting no clear key; an engine without the idle reading getting no clear; a box the clear did not empty named in the error; and the pane's last 15 non-blank lines (or the failed-capture sentence) closing the error.
The former "still pending in the input box after 2 extra Enter(s)" assertion is reworded with the message.
`TestRun_Send_DeliveryVerification`'s never-appeared rows also assert `errors.Is(err, ErrSubmissionNotLanded)`.
`doc.go`'s send paragraph replaces the two-extra-Enter sentence with the settle, the window and the clear.

**ImpactSummary:** `confirmSubmitted` is re-signed with its one caller `sendVerified` in this card, `sendExtraEnters` is deleted with its one user; behaviour changes only for an engine with `InputBoxReader`, which is Claude, on every verified send.

**Commit:** 4: shuttle: settle before Enter, confirm within a window, clear an unlanded send
