# The adopt-check tab closes after the check; reed attach refuses a directory that is no worktree

`reed-tab-and-attach-fixes` · Task · bug, reed · running · Code-Burler-Review

Fix both reed faults in one run.

## From note adopt-check-tab-stays

## Seen (operator, 2026-11-11)

After `lym adopt root` ran its check on OSL-1046, an `adopt-check` tab stayed open at the bottom of the terminal and took a lot of space.
The check had ended green in all three tiers, and the tab was still there.

## Idea

Close the check's pane or tab when the adopt check ends, or shrink it to one line, since its result is already in the adopt record and the creel mail.
Read first which part of `lym adopt` opens the tab, whether a failed check should keep it open for the operator, and where the suite log stays readable after the close.

## From note reed-attach-non-worktree

Imported from [issue #112](https://github.com/Knatte18/loomyard/issues/112).

## What was seen

Running `lym a` (lym reed attach) by mistake in the hub's board checkout `_bolt`, which is no worktree of the hub, booted a tmux session named `_bolt` on the hub's reed socket with one bare shell pane.
`lym reed list` from the prime did not show it; only `tmux ls` did.

## Workaround

`lym reed down` run with cwd `_bolt` removed the session.

## Expected

`lym reed attach` and `lym reed up` refuse a directory that is not a fabric worktree of the hub, such as `_bolt` or a weft checkout, and name the prime or a worktree to attach to instead.

