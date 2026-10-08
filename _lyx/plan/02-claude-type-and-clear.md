# Card 2 — Typing and clearing choreographies on the provider seam

**Create:**
- `internal/shuttleengine#InputBoxClearer`
- `internal/shuttleengine/claudeengine#Claude.TypeSequence`
- `internal/shuttleengine/claudeengine#Claude.ClearInputSequence`
- `internal/shuttleengine/claudeengine#Claude.PastePlaceholder`
- `internal/shuttleengine/claudeengine#pastePlaceholderShape`

**Edit:**
- `internal/shuttleengine#InputBoxReader`
- `internal/shuttleengine/claudeengine/startup_test.go`
- `internal/shuttleengine/claudeengine/session_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/run_test.go`

**Intent:**
Give the provider seam the pieces D2 and D3 need, keeping all Claude TUI knowledge in `claudeengine` (`PATTERN-shuttle-provider-seam`).
Nothing in `shuttleengine` calls them yet; cards 3 and 4 do.

`InputBoxReader` gains `TypeSequence(text string) []PaneInput`: the key choreography that types text as the next turn without submitting it.
Its doc comment says the caller submits with its own Enter once the box has settled, so an engine that reads its box separates typing from submitting (D2).

`InputBoxClearer` is a new optional capability beside `Engine`, declared in `engine.go` next to `InputBoxReader`, with two methods: `ClearInputSequence() []PaneInput`, the key choreography that empties the input box without interrupting a running turn, and `PastePlaceholder(boxText string) bool`, which reports whether an input-box reading is the provider's collapsed paste placeholder.
Its doc comment says the caller sends the clear only into a box it has just read as holding its own text (D3), and that the choreography never uses a key that interrupts a turn.

`Claude.TypeSequence` returns `ComposeSend`'s leading Escape with its `composeSendSettleMS` pause, then the text with `Submit` false and no trailing Enter.
`Claude.ClearInputSequence` returns one `C-u` key: it deletes from the caret to the start of the line, and a sent text is always one line with the caret at its end, so it empties the box; it is neither Escape nor `C-c`, so it never interrupts a turn.
`Claude.PastePlaceholder` matches the box reading against `pastePlaceholderShape`, Claude Code's `[Pasted text #<n>]` / `[Pasted text #<n> +<m> lines]` shape, after trimming.
`session.go`'s compile-time assertion block gains `_ shuttleengine.InputBoxClearer = (*Claude)(nil)`.

Tests: `startup_test.go`'s `TestComposeSend` grows into a table that also pins `TypeSequence` (Escape with its settle, then the text without Submit, and no Enter step); `session_test.go` gains two new test functions, since no existing test covers either method: `TestClearInputSequence` pins the one `C-u` step in `TestClearSessionSequence`'s shape, and `TestPastePlaceholder` tables both placeholder forms, a draft quoting the placeholder text inside other words, and an empty reading.
`run_test.go`'s `inputBoxEngine` fake gains `TypeSequence` (returning `ComposeSend` without its Enter step) so it keeps satisfying `InputBoxReader`.
`claudeengine/doc.go` gains a short paragraph on the split send choreography and the clear.

**ImpactSummary:** One interface method added to `InputBoxReader` (implemented by `Claude` and one test fake) and one new optional capability; no production caller changes behaviour in this card.

**Commit:** 2: claudeengine: type without submitting, clear the input box, read a paste placeholder
