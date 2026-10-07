# Card 2 — reed status line renders wait marks

**Edit:**
- `internal/tokenvocab/tokenvocab.go`
- `internal/tokenvocab/tokenvocab_test.go`
- `internal/tokenvocab/doc.go`
- `internal/reedengine/status-line.md`
- `internal/reedengine/windowsize.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine/waitmark_integration_test.go`

**Uses:**
- `internal/reedengine/io.go`

**Intent:**
Make reed's default status line show every marked pane of the session as `<pane title> ⏳<label> <elapsed>`, with the elapsed time computed by tmux at each status refresh (D2).

The segment is a status-line template token, `{{.waits}}`, so a custom `Config.StatusLine.Template` can place it or drop it.
`tokenvocab`'s registry gains the `waits` token; it resolves to a fixed placeholder string, not to tmux format text, because `pinGeometryOptionsLocked` escapes every `#` in the rendered text (`escapeStatusText`) and would otherwise neutralise the format.
`pinGeometryOptionsLocked` escapes the rendered text as today and then replaces the placeholder with the raw tmux format: a loop over the session's windows and panes (`#{W:#{P:…}}`) that emits, for a pane whose `@lyx_wait` is set, `#{pane_title} ⏳#{@lyx_wait} ` followed by the elapsed minutes computed from strftime's `%s` minus `#{@lyx_wait_start}` (`#{e|-:…}` and `#{e|/:…,60}`), and nothing for an unmarked pane.
`status-left-length` is computed over the escaped text without the placeholder, plus a fixed allowance for the segment, so the identity text is never truncated by the format string's own length.
The default `status-line.md` appends ` {{.waits}}` after the identity text, and its banner comment lists the new token.
An unmarked session therefore renders exactly as before, since the segment expands to nothing.

Open risk named in the decision record: tmux's elapsed arithmetic in a status format must expand on the tmux in use.
The new tmux-tier test (`//go:build tmux && linux`, under the package's existing tmux test main) boots a session, sets a mark through `SetWaitMark`, and asserts that expanding the segment format against that session yields the pane title, the label and an elapsed minute count; if the arithmetic does not expand, the implementation switches the segment to the start time (`since HH:MM`, from `#{t/f/%H:%M:…}` over `@lyx_wait_start`) and the test asserts that form instead, as the acceptance criteria allow.

`tokenvocab_test.go`'s registry tests gain the `waits` row (the all-keys test counts the registry rather than pinning a literal).
`windowsize_test.go` pins, over the fake tmux, that the `status-left` value carries the raw segment format unescaped while the identity text is still escaped, and the `status-left-length` value.
Both `doc.go` files say what the token renders.

**ImpactSummary:** The status-left value and the token registry gain one segment; unmarked sessions render as before and a custom template without the token is unaffected.
