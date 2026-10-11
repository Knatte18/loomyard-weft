# Card 8 — Clear a lone stray character from an idle box once, then read it as residue

**Create:**
- `plan:internal/shuttleengine#StrayInputReader` -> `type StrayInputReader interface`
- `plan:internal/shuttleengine#strayClear` -> `type strayClear struct`
- `plan:internal/shuttleengine/claudeengine#Claude.StrayInput` -> `func (c *Claude) StrayInput(capture string) (text string, ok bool)`

**Edit:**
- `internal/shuttleengine#readiness`
- `internal/shuttleengine#judgeReadiness`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#busyReading` -> `func busyReading(sc sendContext, cycler SessionCycler, hold *turnStartHold, stray *strayClear) string`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/claudeengine/session_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-pane-text-injection.md`

**Intent:**
D8, closing #554.
`StrayInputReader` is an optional capability beside `Engine` with one method, `StrayInput(capture string) (text string, ok bool)`: ok is true when capture shows the provider idle but for an input box holding exactly one stray-character candidate, and text is that character.
`Claude.StrayInput` implements it in `claudeengine`, which alone judges a candidate (`PATTERN-shuttle-provider-seam`): exactly one character that is neither a letter, a digit nor whitespace, in a box located as `IdleSession` locates it, with no running turn on screen.
The lone `%` keeps its own rule and reads empty, so it is never a candidate.

`readiness` gains `draft`, the box text of a reading held for a draft; `judgeReadiness` sets it on the draft hold only.
`awaitIdleSession` hands each send attempt one zero `strayClear`, so the clear fires at most once per attempt.
`busyReading` finds a candidate on both readings a send takes: the hook reading held only for a draft that `StrayInput` reads as the candidate, and the pane-fallback reading of an unknown state whose pane reads ready and not idle while `StrayInput` reads the candidate.
On a candidate before any clear it asks `operatorActivityHold` once: an active operator holds the send, quoting the box text;
otherwise, for an engine with `InputBoxClearer` and `InputBoxReader`, it plays `ClearInputSequence` right after that reading with no wait or other key between, records the character on `strayClear`, sleeps `SubmitSettle`, and takes a second full guard reading by running itself again.
That post-clear reading is the write's guard reading: session ready, no operator activity within `operator_quiet_s`, and the box empty or the residue reading.
The residue reading is shuttle's and applies only on the post-clear reading: a box that still holds exactly the recorded character reads as empty, on the hook path and the pane-fallback path alike, and the send proceeds as for the lone `%`.
Anything else after the clear holds as a draft, quoting the box.
Every pane-fallback hold whose box reads non-empty quotes the box text, keeping the substring "the pane is not idle" the existing rows assert.
Bound: the clear fires only for a candidate, only behind a full guard reading, at most once per send attempt; an operator loses the punctuation character they left longer than `operator_quiet_s`, plus anything typed on that line between the reading and the clear.
A real character that survives the clear and the settle is submitted at the head of the message by the paste-submit's own Enter;
the post-paste confirm keeps its exact-text reading, so a box holding the character beside the sent text fails with `ErrSentTextInDraft`, sends no key and records the pending send, as today.

`PATTERN-pane-text-injection`'s background changes in the four places the decision record's Constraints name: the lead sentence gains the bounded exceptions; the guard bullet's one-fresh-reading clause gains the post-clear reading as the write's guard reading; the lone `%` sentence gains the residue reading, which is shuttle's and taken only after the clear beside the provider's box reading; the explicit key deliveries gain the clear into a box holding a candidate.
The `PATTERN.md` line is unchanged.
`shuttleengine/doc.go`'s guard paragraph and `claudeengine/doc.go` state the rule.

Tests:
`TestSend_WaitsForIdleSession` gains the Stray character rows: a box holding only `+` with no operator activity is cleared and the send goes out with exact text, on the hook reading and on the pane-fallback reading; a lone `+` still on screen after the clear and settle is residue and the send goes out; operator activity in the settle holds at the post-clear reading; a lone `+` with operator activity, and a two-character box, hold with the text quoted.
`TestRun_Send_ConfirmsSubmission` gains a residue send whose post-paste box holds `+` beside the sent text failing with `ErrSentTextInDraft`, sending no key and recording the pending send.
The fake engine in `fakes_test.go` implements `StrayInputReader`.
`StrayInput`'s candidate table is a new top-level test in `claudeengine/session_test.go` (`+`, `-`, `>`, a letter, a digit, `%`, two characters, a running turn), which raises `internal/shuttleengine/claudeengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.

**ImpactSummary:** every verified send of a Claude run, through `awaitIdleSession`, may now clear a lone punctuation character once instead of holding as a draft; engines without `StrayInputReader` keep today's guard, and fallback holds now quote the box.
