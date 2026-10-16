# reed add without --parent splits the selvage shell window

`reed-add-parentless-placement` · Note · bug, reed

Imported from [issue #673](https://github.com/Knatte18/loomyard/issues/673).

Seen: in the hub prime, the orch ran lyx reed add --name stencil-audit --cmd "claude --resume <id> ..." with no --anchor and no --parent. The default anchor is below-parent, and with no parent the new strand ly:stencil-audit was split into tmux window 'shell' (13.0), beside the selvage bash pane (13.1). The shell window must hold only the selvage terminal.

Expected: a strand with no parent never lands in the selvage shell window. With no --parent, reed add either places the strand in its own window (as --anchor own-window does) or refuses below-parent and names --anchor own-window.

Workaround: pass --anchor own-window to every parentless reed add in the prime (to be recorded in CLAUDE.md with this issue number).