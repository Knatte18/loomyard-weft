# Card 1 — Pane listings carry the foreground and start commands

**Edit:**
- `internal/reedengine#LivePane`
- `internal/reedengine#parsePaneList`
- `internal/reedengine#paneListFormat`
- `internal/reedengine#windowPane`
- `internal/reedengine#windowPane.livePane`
- `internal/reedengine#windowPaneFormat`
- `internal/reedengine#parseWindowPanes`
- `internal/reedengine#TestParsePaneList`
- `internal/reedengine#encodeLivePanes`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine#paneTailSeparator`
- `internal/reedengine#splitPaneTail`
- `internal/reedengine#TestParseWindowPanes`

**Uses:**
- `internal/reedengine#windowPaneListingOf`

**Intent:**
D1's condition 3 reads tmux's `#{pane_current_command}` and `#{pane_start_command}` for every bound pane, and `presentPanes` takes a pane from either listing, so both listings carry both fields (D1, condition 2's "one listing change").
`LivePane` gains two string fields, `CurrentCommand` (the pane's foreground process group leader's name) and `StartCommand` (the command tmux froze into the pane at spawn, empty when the session held no `default-command` pin), both `omitempty` in JSON.
`windowPane` gains the same two fields and `windowPane.livePane` maps them onto the `LivePane`.

Both values can hold spaces, as the title can, so the formats keep the six space-split leading fields (paneListFormat) and seven (windowPaneFormat, counting group and own flag as today) and end in one tail field that joins the title, the current command and the start command with the ASCII unit separator `paneTailSeparator` ("\x1f"), which tmux passes through a format verbatim and none of the three values holds:
`paneListFormat` becomes `"#{pane_id} #{pane_dead} #{pane_top} #{pane_width} #{pane_height} #{pane_pid} #{pane_title}\x1f#{pane_current_command}\x1f#{pane_start_command}"`, and `windowPaneFormat` ends the same way after `#{pane_pid} `.
`splitPaneTail` splits the tail field on the separator into title, current and start, missing parts read empty, so a line an older listing or a test fake writes with only a title still parses with both commands empty, which D1 reads as "skip condition 3".
`parsePaneList` and `parseWindowPanes` take the last `SplitN` field through `splitPaneTail` instead of using it as the title.
Neither parser trims the separator: `strings.TrimSpace` strips no `\x1f`, so a trailing empty start command survives.

The test fake: `encodeLivePanes` today writes six fields and no title at all.
When either command is set it appends a tail of the title, `\x1f`, the current command, `\x1f` and the start command; with both commands empty it writes today's six fields and no title, even for a pane whose `Title` is set, so every existing fixture, the title-drift ones included, encodes exactly as today.
`windowPaneListingOf` already copies the whole seventh field verbatim, so it carries both commands into the derived session listing with no change.

Tests: `TestParsePaneList` gains rows for a tail holding a title with spaces, a current command and a start command with spaces (`/bin/bash -l`), a tail with an empty title and both commands, a tail ending in an empty start command, and a line with no separator (title only, both commands empty).
`TestParseWindowPanes` is new, since no test calls `parseWindowPanes` today: it pins the window id, own flag, group, dead flag, pid and the three tail values on a line with all three set, a line with only a title, and a short line that is skipped.

`internal/reedengine/doc.go`'s pane-enumeration paragraph, which names the six-field format and the title as everything after the sixth field, is rewritten for the tail field and its separator.

**ImpactSummary:** Two formats, two parsers and one fake encoder inside reedengine; no caller outside the package reads the format, and existing fixtures stay valid because a tail with no separator parses as the title alone.

**Commit:** 1: reed's pane listings carry the foreground and start commands
