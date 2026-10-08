# Card 5 — Two-line status bar and pane-border titles

**Edit:**
- `internal/reedengine/windowsize.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/attach_test.go`
- `internal/reedcli/smoke_selvage_keepalive_test.go`
- `internal/reedengine/doc.go`

**Create:**
- `internal/reedengine/bar.go`
- `internal/reedengine/bar_integration_test.go`

**Delete:**
- `internal/reedengine/waitmark_integration_test.go`

**Intent:**
Replace the single-line status text with the two-line bar of D4, D5 and D6, and give each strand pane a border title (D6), reading strands only through `@lyx_strands`, `@strand` and `@strand_color` (card 3), never through the current window.

`bar.go` declares the three format strings as package constants with their doc comments, built from tmux 3.6's own default `status-format[1]`/`[2]` shape (list mode, `#{W:}`/`#{P:}`/`#{S:}` loops, `<`/`>` markers):

- Line 1 (`status-format[0]`), list mode (`list=on`, `list=focus` on the lit button, `<` and `>` markers), left to right:
  `VIEW` as a user range (`range=user|view`), lit when the current window carries `@lyx_strands` and is not zoomed;
  for every window carrying `@lyx_strands` (a `#{W:}` loop with a `#{P:}` loop inside), one button per pane carrying `@strand`, in pane order, as a pane range, labelled `@strand`, lit (background `@strand_color`, black, bold) when that pane is the zoomed active pane of its window, else unlit (`@strand_color` on dark grey), white when `@strand_color` is unset, and with ` ⏳<@lyx_wait> <elapsed>m` appended while `@lyx_wait` is set, the elapsed minutes computed by tmux at each refresh as the retired waits segment did (`#{e|/:#{e|-:%s,#{@lyx_wait_start}},60}`);
  when the session has more than one window, one window-range button per window without `@lyx_strands`, labelled with the window name, lit when it is the current window.
- Line 2 (`status-format[1]`): a `#{S:}` loop in its default session-id order (never `#{S/n:}`), one session-range button per session, in list mode with focus on the client's current session; the current session shows its full name lit (yellow, black, bold), every other `#{=12:session_name}`; then `%H:%M` right-aligned.
- `pane-border-format`: ` <@strand> ` in `@strand_color` (white when unset), bold on the active pane, and ` selvage ` for a pane without `@strand`.

Pins (`windowsize.go`): `pinGeometryOptionsLocked` now issues `set-option -g status-format[0]`, `set-option -g status-format[1]` (global only: tmux's status formats are one array, and a per-session index would drop the global lines for that session), `status 2` session-scoped (replacing `on`), `status-position bottom` as today, and, window-scoped on the strand window only, `pane-border-status top` and `pane-border-format`, so a batten window gets no border title; `window-size latest` and the hook unset half stay as they are.
The `status-left`, `status-left-length`, `status-right`, `window-status-format` and `window-status-current-format` pins go, together with the `StatusLineText` call, the waits placeholder swap, `waitsSegmentFormat`, `waitsSegmentLengthAllowance`, `escapeStatusText` and `statusLeftLength`, which only that text needed (the template pipeline itself goes in card 6).
Every pin stays non-fatal (`logger.Warn`, Shared Decision geometry-tmux-failures-are-non-fatal-everywhere) and unbranched on Windows.
The pins still run at the boot that creates the session and in the attach pre-flight, so a server booted by an older lyx picks them up at the next `lyx reed attach`.
`readStatusRowsLocked` already maps a `#{status}` of `2` to two reserved rows, so the layout box shrinks by two rows with no planner change.

Tests:
`windowsize_test.go`'s pin test asserts the new argv set (targets, scopes, values) and that one failing call stops none after it; the rows for the removed text helpers go.
`attach_test.go`'s pin-order case drops its status-text setup and asserts the new pins precede the `#{status}` readback.
`bar_integration_test.go` (`//go:build tmux && linux`, the package's pty attach harness) boots a session with two strand panes, Selvage and a second window, attaches a client, and expands the formats with `display-message -c <client> -p '#{E:status-format[0]}'` (and `[1]`, and `#{E:pane-border-format}` per pane) to assert the acceptance rows: `VIEW` lit exactly when the strand window is current and unzoomed; a strand button lit exactly when its pane is the zoomed active pane; a window button lit when current, also seen from the second window with `VIEW` and the strand buttons still present; no button for Selvage and its border reading `selvage`; a marked pane showing `⏳<label> <n>m` and nothing once cleared; the current session's full name lit and another session cut to 12 characters, in session-id order.
`waitmark_integration_test.go` is deleted: it pinned the retired status-line waits segment, and the wait rows above replace it.
`smoke_selvage_keepalive_test.go`'s status-line subtest asserts `#{status}` reads `2` and `#{status-position}` `bottom` instead of the retired identity text, keeping its Windows self-correcting branch.

`doc.go`: the geometry-option-pins bullet and the `SetWaitMark` paragraph describe the new pins and that wait marks render on strand buttons.

**ImpactSummary:** the status-line pin set changes for every reed session at boot and attach; layout reserves two status rows through the existing readback.
