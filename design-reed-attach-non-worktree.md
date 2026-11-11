# reed: lym a in the hub's _bolt directory boots a reed session there

`reed-attach-non-worktree` · Note · bug, reed

Imported from [issue #112](https://github.com/Knatte18/loomyard/issues/112).

## What was seen

Running `lym a` (lym reed attach) by mistake in the hub's board checkout `_bolt`, which is no worktree of the hub, booted a tmux session named `_bolt` on the hub's reed socket with one bare shell pane.
`lym reed list` from the prime did not show it; only `tmux ls` did.

## Workaround

`lym reed down` run with cwd `_bolt` removed the session.

## Expected

`lym reed attach` and `lym reed up` refuse a directory that is not a fabric worktree of the hub, such as `_bolt` or a weft checkout, and name the prime or a worktree to attach to instead.
