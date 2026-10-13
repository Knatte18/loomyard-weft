# Card 9 — Shuttle tells a draft hold apart from its reason text

**Create:**
- `plan:internal/shuttleengine#busyHold` -> `type busyHold struct`
- `plan:internal/shuttleengine#IsDraftHold` -> `func IsDraftHold(err error) bool`
- `plan:internal/shuttleengine#ErrForeignBoxText` -> `var ErrForeignBoxText = fmt.Errorf("%w: the input box holds text that is not this send's", ErrSessionBusy)`

**Edit:**
- `internal/shuttleengine#IdleProbe`
- `internal/shuttleengine#SessionBusyError`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#busyReading` -> `func busyReading(sc sendContext, cycler SessionCycler, hold *turnStartHold, stray *strayClear) busyHold`
- `internal/shuttleengine#clearStrayCharacter` -> `func clearStrayCharacter(sc sendContext, cycler SessionCycler, hold *turnStartHold, stray *strayClear, character string) (busy busyHold, handled bool)`
- `internal/shuttleengine#withBoxQuote` -> `func withBoxQuote(reason string, engine Engine, capture string) busyHold`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#confirmPasted`
- `internal/shuttleengine#TestRunner_SessionIdle_ReturnsScriptedClassification`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/shuttleengine#TestRun_Send_ConfirmsSubmission`
- `internal/shuttleengine/doc.go`

**Intent:**
D9, shuttle's half: both routes by which draft text reaches the orch carry a draft kind apart from their reason text, through shuttle's provider-neutral types (`PATTERN-shuttle-provider-seam`; Claude's box shape stays in `claudeengine`).
The reason and error texts are unchanged, so loom's `driver_busy` refusal, which renders the send error, still names the draft; `internal/loomcli/resume_test.go` and `internal/battencli/wire_test.go` already pin the refusal naming the draft over an `ErrSessionBusy` error, and `TestSend_WaitsForIdleSession`'s busy rows below still assert the draft text in shuttle's real error.

`IdleProbe` gains `Draft bool`: true when `Idle` is false because the input box holds text, the hold the reading names as a draft.
`Runner.SessionIdle` sets it on the held-reading branch from `readiness.draft != ""`; every other probe leaves it false.

`busyHold` is one idle-wait reading's answer: `reason`, empty when the session is idle, and `draft`, true when the hold is text in the input box.
`busyReading` returns it: `draft` is true for a held hook reading whose `readiness.draft` is set, and for a pane-reading hold whose `withBoxQuote` quoted a non-empty box;
`withBoxQuote` returns the quoted reason with `draft` true when it quoted the box, and the bare reason with `draft` false otherwise;
`clearStrayCharacter` returns a `busyHold` for each of its three answers, with `draft` true for the active-operator hold that quotes the box's character, false for a failed clear, and the post-clear reading's own value otherwise.
`awaitIdleSession` treats an empty `reason` as idle as before and copies `draft` into the `SessionBusyError` it returns.

`SessionBusyError` gains `Draft bool`, set by `awaitIdleSession` from the last reading.
`confirmPasted`'s two box refusals stay errors of their own, never a `*SessionBusyError`, so `pauseengine`'s resume report, which renders a `*SessionBusyError` by its `Reason` and any other error as "the send failed", reads them as it does today:
the sent text beside other text already wraps `ErrSentTextInDraft`;
text that is not the send's now wraps the new sentinel `ErrForeignBoxText`, which wraps `ErrSessionBusy` and spells the same words, so the refusal's message is byte-identical (`<ErrSessionBusy>: the input box holds text that is not this send's: "<box>"`, then the pane tail).
`IsDraftHold` reports a draft hold: a `*SessionBusyError` with `Draft` set (through `errors.As`), or an error that `errors.Is` `ErrSentTextInDraft` or `ErrForeignBoxText`.
`doc.go`'s account of `ErrSessionBusy` and the idle probe names the draft kind.

`TestRunner_SessionIdle_ReturnsScriptedClassification`'s "a draft in the box is held with its reason" row wants `IdleProbe{Reason: …, Draft: true}`; every other row keeps `Draft` false, which the struct comparison already pins.
`TestSend_WaitsForIdleSession` gains a `wantDraft` column checked against `SessionBusyError.Draft` and `IsDraftHold` on every row that fails busy: true for the hook-reading draft rows ("draft in the box fails busy with the reading's reason", "a two-character box holds as a draft on the hook reading", "an unreadable activity query still holds on a draft"), the pane-reading rows that quote a box ("draft that never clears fails busy", "a two-character box holds with the box quoted on the pane reading") and the active operator beside a lone stray character, and false for "a turn is running", the activity-only hold, the failed final capture and the frozen-clock busy pane.
`TestRun_Send_ConfirmsSubmission` gains a `wantDraft` column checked through `IsDraftHold` on every row with a `wantErr`: true for the rows whose error is `ErrSentTextInDraft` or "not this send's", which also add `ErrForeignBoxText` to their `wantIs`, and false for the not-landed rows.

**ImpactSummary:** two exported struct fields, one exported sentinel and one exported predicate in `internal/shuttleengine`; three unexported helpers re-signed with their only callers (`awaitIdleSession`, `busyReading` itself) listed here, and every error text and existing `errors.Is` chain unchanged.

**Commit:** 9: shuttle: carry a draft kind on the idle probe and the busy send error
