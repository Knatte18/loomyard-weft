# Card 42 — orch-role-stencil

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`

**Intent:**
Rewrite the orch role stencil for dobby (D6, D7, D9, D10), with "umbrella" written as "beam" in every text this card touches.
The run loop's first item no longer starts runs: a task is released for its run by `lyx board set-status '{"slug":"<slug>","status":null}'` once the operator approves it, a new task being `draft` until then, and the dobby of the task's parent worktree starts it, drives its batten passes, lands it and cleans up; the orch never starts, stops or cleans up a batten run, and drops `lyx batten run <slug> --window`, the start-on-notice steps and the creel-log fallback.
A halted or paused batten run is resumed only by a hand pass, `lyx batten run <slug>`, which the orch runs only on the operator's word or after judging the stop under Investigating a stop; a halted child is still resumed with `lyx loom resume` in the task worktree as today.
The notices section lists what still reaches the orch: `[batten notice]` lines for a child that halts and whose driver handed the stop back, an awaiting child with a parent notice, the busy-pane draft stall, the owner-elsewhere notice, a driver-dead or api-error notice, a quiet notice (informational), and a Land or Worktree-Teardown failure once the stop chain handed it back; and dobby's own lines from `<shortname>:DOBBY` or `<shortname>:<slug>:DOBBY`: a refused start, a pass that keeps failing, and the board tidy's report path, whose removals and scope changes the orch puts to the operator.
After landing drops the `landed` and `newly ready` bullets and their fallback: a landing that succeeds sends nothing, a records-stop line is the one notice of a successful landing, and a task that reads done is no longer counted running.
Acting without asking says the orch releases the tasks the operator approved instead of starting them, and that `lyx orch start` starts every dobby this machine owns first, warning with `lyx beam start` when that fails.
The stencil's markers and its leading comment are unchanged.
