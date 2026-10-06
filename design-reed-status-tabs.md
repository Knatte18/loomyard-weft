# reed shows strands as zoom tabs and runs as session tabs

`reed-status-tabs` · Note · enhancement, reed, ide

reed runs one tmux session per run on the hub's one server, each with one window named `claude` whose panes are the run's strands, so the default status line shows nothing to switch between and a strand is hard to see on its own.

Chosen by the operator 2026-10-05 after a live demo on a separate server (`tmux -L lyxdemo`, session `run-c`): keep reed's current layout as the overview (driver on top at its fixed height, spawned strands below taking the rest, Selvage one line at the bottom), and show one strand at a time by zooming it.
Everything below is tmux's own options and bindings; reed pins them at server boot, like the `mouse` pin in `internal/reedengine/lifecycle.go`, so they survive a server restart.

- Line 1: a `VIEW` button lit when nothing is zoomed, then one button per strand, lit when that strand is zoomed; Selvage gets no button.
- Strand names come from a pane user option `@strand` that reed sets when it starts the strand (e.g. `driver`, `discussion-writer`), not from `pane_title`, which Claude Code overwrites with its own title; Selvage carries no `@strand`, which is what skips it.
- Line 2: one button per run (session), the current one lit, and the clock.
- Alt+z toggles between the overview and the zoomed strand; Alt+Up/Down moves to the previous/next strand and keeps the zoom; Alt+Left/Right switches run (operator, 2026-10-06).
- Pane borders carry the strand's title.

```
status 2
status-style bg=colour234,fg=white
status-format[0] "#[align=left]#{?window_zoomed_flag,#[bg=colour238#,fg=white],#[bg=cyan#,fg=black#,bold]} VIEW #[default]  #{P:#{?#{@strand},#[range=pane|#{pane_id}]#{?#{&&:#{window_zoomed_flag},#{pane_active}},#[bg=cyan#,fg=black#,bold],#[bg=colour236#,fg=white]} #{@strand} #[norange default] ,}}"
status-format[1] "#[align=left]#{S:#[range=session|#{session_id}]#{?#{==:#{session_name},#{client_session}},#[bg=yellow#,fg=black#,bold],#[bg=colour238#,fg=white]} #{session_name} #[norange default] }#[align=right]%H:%M "
pane-border-status top
pane-border-format " #{?pane_active,#[bold],}#{?#{@strand},#{@strand},selvage} "
bind -n M-z resize-pane -Z
bind -n M-Up select-pane -t :.- -Z
bind -n M-Down select-pane -t :.+ -Z
bind -n M-Left switch-client -p
bind -n M-Right switch-client -n
```

Status formats are one array: setting one index on a single session drops the global lines for that session, so set them globally only.

Open:

- Clicking a strand button while zoomed unzooms (tmux's default `MouseDown1Status` selects the pane); a root binding that re-zooms after the switch when the window was zoomed keeps it zoomed.
- Check that Claude Code does not need Alt+arrows or Alt+z in its input; tmux takes them first.
- VS Code's terminal swallows Alt+arrows and Alt+z, while Konsole passes them through; on 2026-10-06 the operator's user `keybindings.json` got one `workbench.action.terminal.sendSequence` entry per key (`\u001b[1;3A`–`D`, `\u001bz`, `when: terminalFocus`), which made them reach tmux.
  VS Code binds these keys itself (Alt+Left/Right focus the previous/next terminal in a group, Alt+z toggles word wrap), so the terminal never forwards them.
  `lyx ide` seeds these entries as part of its setup (operator, 2026-10-06); VS Code has no per-workspace keybindings, so they go in the user `keybindings.json` as a marked `// lyx:begin` … `// lyx:end` block that `lyx ide` rewrites, as mill does with its own block.
- `switch-client -n`/`-p` steps sessions in name order, but the `#{S:}` loop draws them in session-id order, so Alt+Left/Right jumped around the bar; on 2026-10-06 the bindings were replaced by a `run-shell` helper that steps through `list-sessions` sorted by session id.
  The operator wants spawn order (it matches how the runs sit on their desktop), so the bar keeps session-id order and switching follows it; the helper becomes a lyx verb such as `lyx reed switch --next|--prev` that the keys call.
  The binding passes `--socket '#{socket_path}' --client '#{client_name}'`, so the verb resolves nothing from a working directory: it lists the server's sessions, sorts them by session id and switches that client.
  Session ids are reassigned when the server restarts, so reed recreates the sessions in spawn order (the prime first, then each pair by its run's start time, read from the pair's own records), which keeps the order across a restart.
- Alt+Up/Down cycles every pane, Selvage included; skip it, e.g. by binding to a short reed-free tmux command that steps over panes without `@strand`.
- Extra strands the operator asks for (operator, 2026-10-06): the operator wants extra Claude strands beside the orch in the prime, such as one for exploring, which the orch adds with `lyx reed add --name <name> --cmd claude`.
  Where such a strand goes in the layout relative to the orch, and how several share the space, is open.
  Tried 2026-10-06: `lyx reed add --name explore --cmd claude` runs a bare `claude`, so the pane title is `ly:explore` but the Claude session gets no `--name`, and reed records no session id, so the watchdog's session-name repair (`internal/reedengine/namerepair.go`) skips it.
  Every agent in lyx runs through shuttle (operator), so this needs a shuttle engine extension: open a named LLM session in a strand with no turn and no prompt, started with the strand name and its session id recorded, so the watchdog keeps the name across a compact.
  The verb that calls it sits above shuttle, which has no CLI of its own (see `creel`), and not in reed, which shuttle depends on.

Rejected in the demo: one window per strand (loses the overview, since a pane lives in one window only) and grouped sessions for a second view (hard to attach).

Related: `strand-colors` could tint the strand buttons and borders per role.
