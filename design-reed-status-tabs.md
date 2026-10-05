# reed pins clickable run and strand tabs at boot

`reed-status-tabs` · Note · enhancement, reed

reed runs one tmux session per run on the hub's one server, each with one window named `claude`, so the default status line shows nothing to switch between.

Tried live 2026-10-05, operator approved: the hub set these global options on the reed server, and the operator wants reed to pin them at boot, like the `mouse` pin in `internal/reedengine/lifecycle.go`:

```
status 2
status-format[0] '#[align=left]#{S:#[range=session|#{session_id}]#{?#{==:#{session_name},#{client_session}},#[bg=yellow#,fg=black#,bold],#[bg=colour238#,fg=white]} #{session_name} #[norange default] }#[align=right]%H:%M '
status-format[1] '#[align=left]#{P:#[range=pane|#{pane_id}]#{?pane_active,#[bg=cyan#,fg=black#,bold],#[bg=colour236#,fg=white]} #{pane_index}:#{pane_title} #[norange default] }'
status-style bg=colour234,fg=white
pane-border-status top
pane-border-format ' #{?pane_active,#[bold],}#{pane_index}:#{pane_title} '
```

Line 1 lists every run (session) as a tab, line 2 the strands (panes) of the current run by role; a click switches through tmux's default `MouseDown1Status` binding (`switch-client -t =`).
They are set with `-g`, so every session on the server shows them; set live, they vanish when the server restarts.

Related: `strand-colors` could later tint the pane tabs and borders per role.
