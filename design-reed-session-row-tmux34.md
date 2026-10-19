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

## From issue #748

Imported from [issue #748](https://github.com/Knatte18/loomyard/issues/748).

Fix approach for #738 and #740 together, part of the Ubuntu 24.04 toolchain work in #747.

tmux 3.4's man page says that only `W:` and `P:` take a second, current-item format; `S:` takes one, so reed's `#{S:<other>,<current>}` in `status-format[1]` draws every session twice with a literal comma (#738).
`S:` also walks sessions in name order with no sort option in 3.4, so the prime is not leftmost (#740).

Both go away in one format string that works on tmux 3.4 and newer:
1. Draw the prime's session first, outside the loop.
2. Loop with `#{S:...}` over the others only, skipping the prime with a test such as `#{?#{==:#{session_name},<prime session>},,...}`.
3. Inside the loop, pick the focused style with `#{==:#{session_name},#{client_session}}` instead of a second format, so no comma-separated pair is needed.

Expected: the prime's session leftmost, every other session once to its right, the current one highlighted, on tmux 3.4 and later.
Check it rendered, e.g. with `tmux display-message -p '#{T:status-format[1]}'`, since a test that only checks the option is set does not catch either fault.