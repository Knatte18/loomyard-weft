# Move a running task to another parent (beam or prime)

`reparent-running-task` · Task · feature, batten, board, fabric, workspaces · draft

Imported from [issue #48](https://github.com/Knatte18/loomyard/issues/48).

## Seen

In hub Models (shortname mo), beam duplication-and-import-followups (P2) is nearly done and waits only on one member task and the operator's manual PR review.
The operator wants to start a new piece of work (porting a colleague's rheology and PVT branch) that builds on P2's code but should land through its own project, not grow P2's PR to main.
The natural move is to spawn the task from P2, so it starts on P2's code, and re-parent it to main or to a new beam before it lands, once P2 has landed.
lym has no way to do this:

- A claimed board entry refuses every edit but its status, so its project field cannot move.
- The batten run and the fabric pair record the parent branch at Worktree-Create, and Finalize lands into that parent and takes its parent lock.
- The beam's Beam-Await waits for every member, so P2 cannot land while that task runs.

Hand-editing the board JSON, the batten record and the pair's parent together while a dobby reads them risks a half-changed state, and the orch role forbids those edits.

## Workaround

Make a new beam from main, merge the old beam's branch into it with lym fabric merge-in, and spawn the task there; merge the old beam in again as it changes, and expect conflicts when the old beam lands in main as a squash commit, since git no longer sees its commits.

## Expected

A verb such as 'lym batten reparent <slug> --to <beam|root>' that moves a task, held or not, to another parent in one step: the board project field, the batten and pair parent records, the old beam's member list (so its Beam-Await no longer waits for it) and the new parent's; refused when the new parent's branch does not contain the commits the task branch was forked from, or with a merge of the new parent at its next Parent-Align.