# Board refuses a project change for a finished motif entry whose pair is gone, and batten cancel --remove cannot clear it

`gone-pair-blocks-board-edit` · Note · bug, board, batten

Imported from [issue #52](https://github.com/Knatte18/loomyard/issues/52).

Seen in hub Models-LYMHUB, beam duplication-and-import-followups, 2026-11-04.

The motif task knut-rheology-friction-pvt finished: lym batten status reports state done, its last row is Land, and the board entry is back to draft with discussed: true. lym fabric pairs lists no pair for the slug and the worktree directory is no git repository; only stray build output (obj directories of two projects) is left under its path.

Moving the entry to another project is refused:
lym board upsert '{"slug":"knut-rheology-friction-pvt","project":"knut-rheology"}'
-> entry has a pair, and a change of its project moves its pair out from under its project's beam: end the pair first with lym batten cancel knut-rheology-friction-pvt --remove, then change project

The way forward it names fails:
lym batten cancel knut-rheology-friction-pvt --remove
-> battencli: could not read the task worktree of knut-rheology-friction-pvt: not a git repository

Expected: a finished run whose pair is torn down leaves no pair record that blocks a board edit; or batten cancel --remove clears the record when the worktree is already gone, instead of requiring a readable git worktree. Also, the stray obj directories under the removed worktree's path suggest something built there after teardown.

Workaround: none; the entry stays in its old project.