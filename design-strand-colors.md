# Tell runs and strands apart: tabs, pane titles and colors

`strand-colors` · Note · enhancement, reed, loom, shed, shuttle

## Problem

With a run live, the driver pane and the agents it spawns (Webster, Burler, Bouncer judge, Describe and the rest) look alike in tmux and in Claude Code, so it is often hard to see which session is which.

## Idea (operator, 2026-10-05)

Give each spawned session a color with Claude Code's `/color` command, so the driver stands apart from the agents it spawns.
Open question: one color per loom segment (Discussion, Plan, Webster, review, Describe, landing) rather than only driver against the rest; the operator would like that.

## Open

- Where the color is set: by the spawning code through reed when it starts the strand, or by the agent's own launch prompt.
- Whether the tmux pane border or status line should carry the same color, so the reed layout matches the sessions.


## Runs and strands as clickable tabs (tried live 2026-10-05, operator approved)

reed runs one tmux session per run on the hub's one server, each with one window named `claude`, so the default status line shows nothing to switch between.
The hub set these global options live on the reed server, and the operator wants reed to pin them at boot, like the `mouse` pin in `internal/reedengine/lifecycle.go`:

```
status 2
status-format[0] '#[align=left]#{S:#[range=session|#{session_id}]#{?#{==:#{session_name},#{client_session}},#[bg=yellow#,fg=black#,bold],#[bg=colour238#,fg=white]} #{session_name} #[norange default] }#[align=right]%H:%M '
status-format[1] '#[align=left]#{P:#[range=pane|#{pane_id}]#{?pane_active,#[bg=cyan#,fg=black#,bold],#[bg=colour236#,fg=white]} #{pane_index}:#{pane_title} #[norange default] }'
status-style bg=colour234,fg=white
pane-border-status top
pane-border-format ' #{?pane_active,#[bold],}#{pane_index}:#{pane_title} '
```

Line 1 lists every run (session) as a tab, line 2 the strands (panes) of the current run by role; a click switches through tmux's default `MouseDown1Status` binding (`switch-client -t =`).
Strand colors could later tint the pane tabs and borders per role.
