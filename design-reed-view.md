# reed: strand and run tabs, narrow-terminal run row, a color per strand role

`reed-view` · Task · enhancement, reed · running · Discussion-Burler

The run view in reed: strands as zoomable tabs, runs as session tabs that fit a narrow terminal, and a color per strand role.

## Strand and run tabs

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

Rejected in the demo: one window per strand (loses the overview, since a pane lives in one window only) and grouped sessions for a second view (hard to attach).

Related: `strand-colors` could tint the strand buttons and borders per role.


## Run tabs on a narrow terminal

## Seen 2026-10-06

With five runs and the prime open, the session row of the status line (the run tabs from `reed-status-tabs`) is wider than a narrow terminal, and the rightmost runs fall off the edge with no sign that they exist.
The row prints every session's full name with padding, and run slugs such as `test-suite-prune-core` are long.

## Options

1. Current session full, others truncated, overflow scrolls (recommended): the session the client is in shows its full name, every other session is cut to about 12 characters (`#{=12:session_name}`), and the row uses tmux's list mode (`list=on`, `list=focus`, left and right markers), so a row that still overflows scrolls with `<` and `>` markers and keeps the current session visible.
2. Index only for other sessions, the current one by full name: the most compact, but the operator has to remember which number is which run.
3. List-mode scrolling alone, full names kept: the least change, with the fewest sessions visible at once.
4. Wrap the session row onto further status lines: costs terminal height, and tmux has no automatic wrap, so reed would compute the split itself.

The run tabs' design is in the note `reed-status-tabs`.


## A color per strand role

## Problem

With a run live, the driver pane and the agents it spawns (Webster, Burler, Bouncer judge, Describe and the rest) look alike in tmux and in Claude Code, so it is often hard to see which session is which.

## Idea (operator, 2026-10-05)

Give each spawned session a color with Claude Code's `/color` command, so the driver stands apart from the agents it spawns.
Open question: one color per loom segment (Discussion, Plan, Webster, review, Describe, landing) rather than only driver against the rest; the operator would like that.

## Open

- Where the color is set: by the spawning code through reed when it starts the strand, or by the agent's own launch prompt.
- Whether the tmux pane border or the strand tabs of `reed-status-tabs` should carry the same color, so the reed layout matches the sessions.
