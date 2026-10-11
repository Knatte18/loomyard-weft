# Card 6 — shuttle's paste path and the exact-match Enter guard

**Create:**
- `plan:internal/shuttleengine#TextPaster` -> `type TextPaster interface`
- `plan:internal/shuttleengine#ErrSentTextInDraft` -> `var ErrSentTextInDraft error`
- `plan:internal/shuttleengine#pasteVerified` -> `func pasteVerified(sc sendContext, paster TextPaster, reader InputBoxReader, text string) error`
- `plan:internal/shuttleengine#confirmPasted` -> `func confirmPasted(sc sendContext, paster TextPaster, reader InputBoxReader, text string, sentAt int64, closeAt time.Time) error`
- `plan:internal/shuttleengine#guardedEnter` -> `func guardedEnter(sc sendContext) (sent bool, err error)`
- `plan:internal/shuttleengine#tuiReed.PasteSubmit` -> `func (r *tuiReed) PasteSubmit(guid, text string, submit bool) error`
- `plan:internal/shuttleengine#pasteEngine` -> `type pasteEngine struct`
- `plan:internal/shuttleengine#pasteEngine.PasteSubmits` -> `func (e *pasteEngine) PasteSubmits() bool`
- `plan:internal/shuttleengine#pasteEngine.OwnPastePlaceholder` -> `func (e *pasteEngine) OwnPastePlaceholder(boxText, text string) bool`

**Edit:**
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#boxHoldsSentText` -> `func boxHoldsSentText(reader InputBoxReader, capture, normalized string) bool`
- `internal/shuttleengine#inputBoxHoldsSentText` -> `func inputBoxHoldsSentText(reed ReedOps, reader InputBoxReader, guid, normalized string) bool`
- `internal/shuttleengine#settleAndConfirm` -> `func settleAndConfirm(sc sendContext, reader InputBoxReader, normalized string, sentAt int64, closeAt time.Time) error`
- `internal/shuttleengine#confirmSubmitted` -> `func confirmSubmitted(sc sendContext, reader InputBoxReader, normalized string, sentAt int64, closeAt time.Time) error`
- `internal/shuttleengine#failUnlanded` -> `func failUnlanded(sc sendContext, reader InputBoxReader, normalized string, reason error) error`
- `internal/shuttleengine#clearUnlandedText` -> `func clearUnlandedText(sc sendContext, reader InputBoxReader, normalized string) (landed bool, note string)`
- `internal/shuttleengine#typeColor`
- `internal/shuttleengine#InputBoxClearer`
- `internal/shuttleengine#ErrSubmissionNotLanded`
- `internal/shuttleengine#TestRun_Send_ConfirmsSubmission`
- `internal/shuttleengine#TestRun_Send_DeliveryVerification`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/shuttleengine#isOwnText`

**Uses:**
- `internal/shuttleengine#ReedOps`
- `plan:internal/shuttleengine#operatorActivityHold`

**Intent:**
Route a provider that declares the paste path through reed's atomic paste-submit, and send an Enter of lyx's own only into a box holding exactly the sent text (decisions D2, D5, D6).

`TextPaster` is a new optional capability beside `Engine`, with two methods:
`PasteSubmits() bool` — whether an Enter right after the paste end submits it, so shuttle pastes with the trailing Enter (true) or without it (false, D2's fallback);
and `OwnPastePlaceholder(boxText, text string) bool` — whether `boxText` is exactly the collapsed-paste placeholder a paste of `text` produces (the provider owns the collapse threshold and the line-count shape, `PATTERN-shuttle-provider-seam`).
`InputBoxClearer` loses `PastePlaceholder`, whose only caller was the clear this card makes exact; claudeengine's method stays until card 9 folds it into `OwnPastePlaceholder`.

`ErrSentTextInDraft` is `fmt.Errorf("%w: the sent text sits in the operator's draft", ErrSessionBusy)`, so every caller retrying `ErrSessionBusy` keeps working (D7).

`sendVerified` keeps `awaitIdleSession` first.
For an engine that implements `TextPaster`, `InputBoxReader` and `SessionCycler` it then calls `pasteVerified` and returns its result; any other engine keeps today's typing path.
`pasteVerified` opens the submit window (`submitConfirmTimeout`, cut to the deadline), records `sentAt := eventsSize`, calls `reed.PasteSubmit(guid, text, paster.PasteSubmits())` once with no Escape and no typed keys before it, and hands over to `confirmPasted`; with `PasteSubmits` false it first runs `awaitSettledBox`, so the guarded Enter lands on a redrawn box.

`confirmPasted` reads the box at the `confirmSubmitted` cadence (first read `reader.SubmitSettle()` after the paste, doubling to `confirmBackoffCap`, cut to the window) and decides on each read:
- a turn start past `sentAt`, or an empty box: landed, return nil;
- the box normalizes to exactly the sent text, or `OwnPastePlaceholder(boxText, text)` holds: the Enter did not take, so `guardedEnter` and read again;
- no readable box: send nothing and read again; a window that ends this way fails with `ErrSubmissionNotLanded` and no Enter and no clear key;
- the box contains the normalized sent text beside other text: return an error wrapping `ErrSentTextInDraft` that quotes the box, with no Enter, no clear and no edit;
- any other text: return a plain error wrapping `ErrSessionBusy` that quotes the box it read (`the input box holds text that is not this send's: %q`), with no key sent, so loom's `driver_busy` refusal (card 13) and batten's report (card 14) can name the draft (D8, D9).
A window that ends on an exact match fails with `ErrSubmissionNotLanded` through `failUnlanded`.
`guardedEnter` asks `operatorActivityHold` immediately before the key and sends `Enter` only when it returns ""; a held Enter is skipped and the read repeats inside the window.

Exact match everywhere (D5): `boxHoldsSentText` drops the needle and answers `normalizePaneText(box) == normalized`, so a text of `sendNeedleRunes` or more characters is no longer accepted by a box merely containing its first 48 characters; `inputBoxHoldsSentText`, `settleAndConfirm`, `confirmSubmitted` and `failUnlanded` drop the needle with it.
`clearUnlandedText` clears only a box that is exactly this send's text, re-read immediately before the clear key, and `isOwnText`, with its multi-copy and prefix acceptance, is deleted.
`sendNeedle` stays for the appearance scan of the non-paste path.
`typeColor` follows the re-signed `boxHoldsSentText` and `confirmSubmitted`; card 8 moves it onto the guarded path.

Bound: the placeholder acceptance can submit an operator paste that replaced a swallowed paste of ours; it is limited to a text the provider collapses, a placeholder whose line count matches, a box that read empty before the write, the send's own window and a passing activity check before each guarded Enter (R3).

Fakes: `tuiReed.PasteSubmit` puts `text` into its box and submits it unless the Enter is swallowed, recording through `fakeReed.PasteSubmit`; `pasteEngine` embeds `tuiEngine`, answers `PasteSubmits` from a field and `OwnPastePlaceholder` for a `[Pasted text #1]` box when the text is 20 characters or longer.

Tests:
- `TestRun_Send_ConfirmsSubmission` rows over `pasteEngine` and `tuiReed`: a send issues one `PasteSubmit` and no `SendText`, no `Escape`; a swallowed Enter leaves the exact text, which gets one guarded Enter and lands; a placeholder box gets the guarded Enter only for a long text, and for a short text fails; an active operator before the guarded Enter gets none; a box holding the sent text plus a draft fails wrapping `ErrSentTextInDraft` and `ErrSessionBusy` with no Enter, no `C-u`; a foreign draft fails wrapping `ErrSessionBusy` with no key and an error text carrying the draft; a box-less capture through the window fails `ErrSubmissionNotLanded` with no Enter and no `C-u`; the `PasteSubmits` false variant settles, then sends the Enter.
- `TestRun_Send_DeliveryVerification`: a text of 60 characters whose first 48 sit inside a longer draft is no longer accepted as held.
- `TestStartup_SendFailuresTearDown` rows: a skill load or prompt delivery held past `send_ready_timeout_s` by an active operator, and a prompt delivery failing on a foreign box, each return a died start whose error names the cause.

Raise `internal/shuttleengine`'s row in `cmd/lyx/testdata/test-budget.yaml` if a new top-level test is needed, and rewrite the verified-send section of `internal/shuttleengine/doc.go` for the paste path and the exact-match guard.

**ImpactSummary:** The verified send's confirmation helpers are re-signed inside shuttleengine only; every caller is in send.go or run.go and listed here.
