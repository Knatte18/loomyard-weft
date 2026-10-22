---
verdict: APPROVED
---

## Scope against the board entry

The entry (issue #776) asks for three things, and the decision record covers each of them, no more and no less.

- Name the checked tree and the dirty paths: D2 carries the code checkout's path and its sorted, worktree-relative tracked dirty paths on `MergeGuardError` as typed fields, and Finalize's parent-side stuck reason renders them with a way forward.
  `Error()` stays path-free under the package's closed reason set, and the other surfaces of the bare text are named Out with a reason.
  The entry's title and incident are Finalize's halt, so limiting the rendering to that surface is within the goal.
- Keep lyx's run records from blocking the merge: D1 makes no code change because the premise does not hold for the guard.
  I checked it: `pairDirtyReason` (`internal/fabricengine/mergeguards.go:109`) reads only the warp checkout, and the comment there states that the weft side is not evaluated.
  The entry's second option, committing the records before the check, is what Finalize's step 2b already does.
  The acceptance criterion pins this behaviour, and D3 removes the halt-note section whose both-checkouts read showed the run record as dirt.
- Do not re-step Finalize on an unchanged failure: D4 makes the goto guard refuse Finalize, and every row whose `OnDone` routes to it, while the parent's code checkout has tracked dirt, so a resume hands back and writes no second halt note.
  The open risk, one tackler spawn per dirty-parent halt, stays within "halt once".

Nothing in the design belongs to another entry or takes a direction the entry rules out.

## Board entries that touch this task

- `loop-exit-halt-names-its-cause`: it changes the halt note's reason and anomaly kind for a Merge-In loop exit.
  D3 removes the `parent fabric status:` section and `ParentStatus`.
  Both touch `haltnote` and `loomcli/halt.go`, but at different places and with compatible edits, so this is a text-only overlap.
- `run-verbs-shared`: it plans to move the `lyx loom` run verbs that way-forward texts name.
  D2's way forward uses `lyx loom goto` and `lyx loom resume` under the current convention, and that entry already covers rewriting such texts.
  The two entries do not contradict each other.
- `batten-cancel-verb`: its dirty-source refusal is batten's start precondition, a different guard on a different tree.
  It does not overlap with this task.

## Peers and parent commits

- `run-process-ownership` (row 3, own_row 3): no changed paths and no decision record yet.
  Its goal, process ownership, shares no behaviour with this design.
  No overlap; the rows are a tie.
- `board-entry-handles` (row 0): its decision record changes only `boardengine`, `boardcli` and replay.
  No overlap; the order is unknown.
- Parent commits: none listed.

No overlap needs a landing order.
