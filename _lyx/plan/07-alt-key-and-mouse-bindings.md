# Card 7 — Alt-key and status-bar mouse bindings

**Edit:**
- `internal/reedengine/panebin.go`
- `internal/reedengine/panebin_test.go`
- `internal/reedengine/windowsize.go`
- `internal/reedengine/windowsize_test.go`
- `internal/reedengine/paneoptions.go`
- `internal/reedengine/paneoptions_test.go`
- `internal/reedengine/probe.go`
- `internal/reedengine/probe_test.go`
- `internal/reedengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-pane-binary-resolution.md`

**Create:**
- `internal/reedengine/bindings.go`
- `internal/reedengine/bindings_test.go`
- `internal/reedengine/bindings_integration_test.go`

**Uses:**
- `internal/reedengine/switch.go`
- `internal/reedengine/bar.go`

**Intent:**
Bind the five Alt keys of D8 and replace `MouseDown1Status` per D9, server-wide, at boot and in the attach pre-flight, so a server booted by an older lyx gets them at the next `lyx reed attach`.

`bindings.go` holds a pure builder returning the `bind-key` argvs and a `(e *Engine) pinBindingsLocked()` that issues them, each non-fatal (`logger.Warn` on failure, unbranched on Windows).
`pinGeometryOptionsLocked` calls it, so it rides both existing call sites.
The bindings, all in the root table (`bind-key -n`):

- `M-z`: `if-shell -F` on `#{||:#{@strand},#{window_zoomed_flag}}` → `resize-pane -Z`; otherwise nothing, so Selvage and a batten window are untouched.
- `M-Up` / `M-Down`: only when the current window carries `@lyx_strands`, `select-pane -Z` to the previous/next pane (`:.-` / `:.+`, which wrap), then, when the landed pane has no `@strand`, one more step the same way; Selvage is always the one pane without `@strand`, so one step over it suffices and no reed process runs.
- `M-Left` / `M-Right`: `run-shell -b` of the switch command (below) with `--prev` / `--next`; `-b` because `run-shell` without it blocks the tmux server (`doc.go`'s run-shell bullet).
- `MouseDown1Status`: dispatch on `#{mouse_status_range}`: `view` selects the strand window and unzooms it; a pane range selects that pane in its window and zooms it (zoom kept on, never toggled off); a window range selects that window; a session range switches the client to that session.
  To address the strand window from a binding, `markStrandWindowLocked` (`paneoptions.go`) also records its window id in the session user option `@lyx_strand_window`, and the `view` branch targets it through a format-expanded command (`run-shell -C`).

Bound: the bindings take these five keys and the left status click from every pane on the hub's reed server, so a program in a pane, Claude Code included, never receives Alt+z, Alt+Up/Down or Alt+Left/Right; no other key or mouse binding is changed, and tmux's right-click menus keep their defaults.

The switch command (D11) is composed in `panebin.go` beside `composeWindowCommand`, from the `executablePath` chokepoint, so the binding names the absolute path of the `lyx` that pinned it, never `PATH`'s.
Its tokens go through `shell.ForGOOS()` (`PATTERN-shell-mechanics-seam`): `Invoke(lyx)`, then `reed switch --next|--prev --socket <Quote(socket path)> --client #{q:client_name} --tmux <Quote(tmux path)>`.
Every pinned token has each `#` doubled after shell quoting, since `run-shell` format-expands its whole command at key press; `#{q:client_name}` stands as its own unquoted word, the one value tmux expands at key press.
The socket path is read back with `display-message -p '#{socket_path}'`; the tmux path is `e.cfg.Tmux` resolved to an absolute path with `exec.LookPath`.
An unresolvable executable, socket path or tmux path logs a named `logger.Warn` and pins neither switch binding, leaving the other bindings in place.

Probe: `probe.go` gains an optional-verb list (`bind-key`, `run-shell`, `if-shell`, `switch-client`, beside the already-optional `set-hook` and `resize-pane`) checked after the required set; a missing optional verb logs one `logger.Warn` naming it and never fails the probe, so a psmux without them still boots (the Windows degrade).

PATTERN: `PATTERN-pane-binary-resolution` widens its scope to reed's key bindings that run `lyx` (the one-line entry in `PATTERN.md` and the background file's scope bullet), in this commit.

Tests:
`bindings_test.go` (untagged) pins the builder's argvs: the five keys and the mouse dispatch, `-n` on each, and no other key.
`panebin_test.go` pins the switch command for a `lyx`, socket and tmux path each containing `#` and a single quote: quoted once, every `#` doubled, `#{q:client_name}` unquoted, and the unresolvable-executable degrade.
`windowsize_test.go` asserts the pin path issues the bindings after the option pins; `paneoptions_test.go` asserts the marker also sets `@lyx_strand_window`; `probe_test.go` gains a row where an optional verb is missing and the probe passes with one warning.
`bindings_integration_test.go` (`//go:build tmux && linux`, the pty attach harness) writes the terminal's own byte sequences into the client pty (`ESC z`, `ESC [1;3A`/`B`/`C`/`D`, SGR mouse clicks at the bar's cells) and asserts: Alt+z toggles a strand's zoom and does nothing on Selvage or in a second window; Alt+Up/Down step through strand panes only, wrapping and keeping the zoom; clicks on `VIEW`, a strand button, a window button and a run button each have their D9 effect; and, with `executablePath` pointed at a recorder script under a directory containing `#`, Alt+Right runs it with the socket and tmux paths unchanged.

`doc.go` documents the bindings, their bound, and the optional verbs.

**ImpactSummary:** every reed boot and attach pins six server-wide bindings; the probe warns on optional verbs; PATTERN-pane-binary-resolution widens.
