# reed status bar draws the current session twice with a stray comma on tmux 3.4

`reed-session-row-tmux34` · Note · bug, reed

Imported from [issue #738](https://github.com/Knatte18/loomyard/issues/738).

Seen on Linux with tmux 3.4, in the prime's reed session `loomyard` (the only session on the socket).

The session row (`status-format[1]`) reads ` loomyard , loomyard `: one grey copy, a literal comma, then the yellow focused copy.

Rendered with `tmux display-message -p '#{T:status-format[1]}'`:

```
#[range=session|$0 bg=colour236 fg=white] loomyard #[norange list=on default] ,#[range=session|$0 list=focus bg=yellow fg=black bold] loomyard #[norange list=on default]
```

The format is `#{S:<other-session arm>,<current-session arm>}`.
tmux 3.4 does not split `#{S:}` into a separate current-session format, so every session gets both arms and the comma between them.

Expected: each session is drawn once, the current one highlighted, on every tmux version lyx supports.
For example, use one arm that picks the style with `#{?session_attached,...}` or `#{==:#{session_name},#{client_session}}`, or detect the tmux version.