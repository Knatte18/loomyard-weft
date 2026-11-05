# Board status of a motif run stays at Worktree-Create through Discussion

`motif-board-status-stale` · Note · bug, board, batten, loom

Imported from [issue #49](https://github.com/Knatte18/loomyard/issues/49).

## Seen

Hub Models (shortname mo), beam duplication-and-import-followups, task knut-rheology-friction-pvt (T2), recipe motif, released 2026-11-04.
Some 15 minutes after release, its Discussion-Write session (mo:knut-rheology-friction-pvt:discussion) was live and writing the decision record, and the operator was talking to it.
At the same time:

- the board entry, and so the Bolt README, said running · Worktree-Create;
- lym batten status showed current_producer Run-Shed, history_length 5 (Worktree-Create, Seed-Child, Open-Window, Seed-IDE, Open-IDE done);
- lym loom status in the task worktree showed current_producer empty and history_length 0;
- lym beam status showed the run with an empty producer column.

A tapestry run in the same beam (server-based-live-viewer) showed its inner producer on the board, such as running · Webster.

## Expected

The board status follows the batten rows past Worktree-Create and then the inner run's current producer, Discussion-Write here, and lym loom status names the producer in flight rather than an empty one, so the operator does not read a working run as stuck.

## Issue comment, 2026-11-04

Clarification: the README is rendered by the board module on every board commit, so the README is not at fault. The stale value is the board entry's own status field: nothing wrote it after Worktree-Create, so no board commit re-rendered the README. The fix belongs where the run writes its status to the board entry as it moves through the batten rows and into the inner run's producers.

## Issue comment, 2026-11-04

Board history for the entry, from the board.json commits in Bolt (times +01:00):

- 18:29:56 draft
- 18:30:08 to 18:34:43 running · Worktree-Create (four commits)
- 18:44:31 running · Discussion-Bouncer
- 18:45:01 running · Discussion-Burler-Review

So the status was never written for Seed-Child through Run-Shed, nor for Discussion-Write, which ran for about ten minutes in between; it jumped from Worktree-Create straight to Discussion-Bouncer. lym board get read the same stale value as board.json, so this is the status write, independent of how the README renders it; my earlier comment's claim about the renderer was not checked and should be disregarded.