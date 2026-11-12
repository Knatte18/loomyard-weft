# Study what a re-parent of a running task has to write

`reparent-running-task` · Task · research, board, batten, fabric, dobby · running · Worktree-Create

Imported from [issue #48](https://github.com/Knatte18/loomyard/issues/48).

## Seen

In hub Models (shortname mo), a beam was nearly done and waited only on one member task and the operator's manual PR review.
The operator wanted to start new work that builds on that beam's code but lands through its own project.
The natural move is to spawn the task from the beam, so it starts on the beam's code, and re-parent it to main or to a new beam before it lands.
lym has no way to do this:

- A claimed board entry refuses every edit but its status, so its project field cannot move.
- The batten run and the fabric pair record the parent branch at Worktree-Create, and Finalize lands into that parent and takes its parent lock.
- The beam's Beam-Await waits for every member, so the beam cannot land while that task runs.

The workaround today is a new beam from main, the old beam merged in with `lym fabric merge-in`, and the task spawned there.

## Idea (operator, 2026-11-12)

Pause the task, move every pointer to the new parent, and resume it.
The operator does not see why this is hard, so this task finds out what has to be written, and writes none of it.

## What it does

A read-only study of the code, reported under `docs/research/`:

1. List every place that records a task's parent or its membership of a project or beam, with the file and function that writes it and the one that reads it:
   the board entry, the batten run record, the fabric pair and its records branch, the project and beam member lists, the parent lock, the dobby's view of the run, and anything else the code shows.
2. For each, say whether a pause already holds its writer still, whether a single verb could change it atomically with the others, and what a half-done change would leave behind.
3. Settle the git side: what lands when a task forked from one beam is re-parented to main or to another beam, when the new parent does not hold the commits the task branch started from.
   Compare the two answers the note names (refuse, or merge the new parent at the next Parent-Align) and recommend one.
4. Say how Beam-Await and the old beam's PR behave once the member is gone, and the same for the new parent.
5. End with the list of what a verb `lym batten reparent <slug> --to <beam|root>` would have to write, the order, the refusals, the tests it needs, and the size of the change as a recipe and profile.

## Cap

Reading only; one report; no code change, no run of the verbs on a live task.

## Not in scope

Writing the verb.
