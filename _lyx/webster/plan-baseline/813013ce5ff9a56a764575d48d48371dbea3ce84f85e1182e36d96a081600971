# Card 4 — claude-restored-prompt-clear

**Create:**
- `internal/shuttleengine#RestoredPromptClearer`
- `internal/shuttleengine/claudeengine#Claude.RestoredPromptClearSequence`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.ClearInputSequence`
- `internal/shuttleengine/claudeengine#TestClearInputSequence`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#Claude.PlainBoxCapture`
- `internal/shuttleengine/claudeengine#readPaneFixture`
- `internal/shuttleengine/claudeengine/testdata/pane-restored-prompt-styled.txt`
- `internal/shuttleengine#InputBoxClearer`

**Intent:** Decision D3, provider half: Claude declares the clear that empties a prompt its interrupt restored into the box, its own sequence beside `ClearInputSequence`, which stays one `C-u`.

`shuttleengine` declares, in `engine.go` beside `InputBoxClearer`, the optional capability `RestoredPromptClearer` with the one method `RestoredPromptClearSequence(capture string) []PaneInput`: the keys that empty the input box `capture` shows, sized from that capture, for a box holding the prompt of a turn lyx's own interrupt just stopped.
Its doc states the contract the caller relies on: the sequence holds no Escape, `C-c` or Enter, only deletes box text, and is at most two keys per box line the capture shows.

`Claude.RestoredPromptClearSequence` counts the box's lines as `inputBoxInterior` locates them in `capture`: once the caret marker, side bars and whitespace are stripped from each interior row, n is the number of rows from the first row where something remains to the last, blank rows between them included (last minus first plus one), since a blank row of a multi-paragraph prompt costs a `C-u` and a `BSpace` like any other.
For n lines it returns n `C-u` with one `BSpace` between each pair (`C-u`, then `BSpace`, `C-u` for each further line), and no keys for a box with no such row or no box at all; every counted row is a box line the capture shows, so the sequence keeps the bound of at most two keys per box line.
The live capture taken at plan time pins the shape: the restored prompt is drawn unstyled with the caret at its end, one `C-u` deletes only the caret's row of a wrapped prompt, a `BSpace` at a row's start joins it to the row above, and `C-u`, `BSpace`, `C-u`, `BSpace`, `C-u` emptied a three-row prompt.
`Claude.ClearInputSequence` keeps its one `C-u`; its doc gains that a restored multi-line prompt takes `RestoredPromptClearSequence` instead, and `claudeengine/doc.go`'s `ClearInputSequence` sentence gains the restored-prompt clear beside it.

`TestClearInputSequence` keeps its one-`C-u` assertion and gains a table over `RestoredPromptClearSequence`:
`PlainBoxCapture` of the restored-prompt fixture, three rows, answers `C-u BSpace C-u BSpace C-u`;
a two-row box answers `C-u BSpace C-u`;
a three-row box whose middle row is blank answers `C-u BSpace C-u BSpace C-u`;
a one-row box answers one `C-u`;
an empty box and a capture with no box answer no keys;
and every answer holds no `Escape`, `C-c` or `Enter`.

**ImpactSummary:** Adds a capability and Claude's sequence, which nothing calls until card 5; ClearInputSequence keeps its keys, so the stray-character and sent-text clears are unchanged.
