# A paused run whose work already landed is never finished and keeps its worktree and terminal

`landed-run-left-paused` · Task · bug, dobby, batten · running · Worktree-Create

Depends on: [`batten-stray-dir-and-beam-mark`](design-batten-stray-dir-and-beam-mark.md)

## Seen (2026-11-02)

Run `cold-gate-fixer` ran on osl-1046 and its work reached `main` (`a9536680`), but its batten run never finished Land and Worktree-Teardown there.
After `lym adopt root` took the hub over to this machine, the run stood `paused` at Run-Shed with pause reason "pause requested", with its task worktree and its loom-status terminal still up, for hours.
No notice reached the orch, and dobby does not resume a paused run, so nothing finished it; the operator noticed the open terminal.
`lym batten cancel cold-gate-fixer --remove` refused, saying the work had reached its parent, and named `lym batten run cold-gate-fixer`, which then finished Land and Worktree-Teardown at once.

Whether the pause came from the adopt hold or from the orch pausing the adopted runs is not known.

## Idea

- A run whose work already reached its parent has nothing left to decide: its remaining rows are Land and Worktree-Teardown.
  Dobby finishes such a run even when it is paused, or at least sends the orch a notice naming it.
- `lym adopt` lists the adopted runs whose work already landed and finishes them as part of the takeover.
