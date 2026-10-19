# reed status bar orders sessions by name instead of putting the prime first

`reed-session-row-prime-first` · Note · bug, reed

Imported from [issue #740](https://github.com/Knatte18/loomyard/issues/740).

Seen on Linux, tmux 3.4, with the prime's reed session `loomyard` and two task-pair sessions `hub-workspaces` and `implementer-seam`.

The session row (`status-format[1]`, `#{S:...}`) draws the sessions in tmux's name order: hub-workspaces, implementer-seam, loomyard.

Expected, per the operator: the prime's session always sits leftmost, and the task-pair sessions follow to its right.
Separate from #738, which is the doubled entry with a stray comma in the same row.