# Move a running task by hand-back and a fresh run, not a re-parent verb

`move-task-by-handback` · Note · feature, batten, loom, board

## Seen

The study `docs/research/loom/reparent-running-task-2026-11-12.md` (commit `5e41ee2b`) finds that a verb moving a running task to another parent is feasible but wide:
two durable stores written by two modules with no atomic change, a narrow board write past the run lock,
an ancestry check on the new parent, and a cross-module tapestry run of its own.

## Idea (operator, 2026-11-15)

Moving a running task is rare, and a verb for it is clumsy for what it buys.
Instead, end the run and start a new one with more information:

1. The run's live writer writes back to the task's board entry everything it has learned:
   what is done, the decisions taken, what is left, and where its commits are.
2. The run is cancelled, and its commits are kept under an archive tag, so the new run can read them.
3. A new task, under the new parent, starts from that body.

What the design must settle:

- Who writes the hand-back, since a held entry takes no body edits from a run: a note the operator merges, or a new verb.
- Whether the new run re-uses the old commits (a cherry-pick, which brings back the study's ancestry question) or only reads them.
- Whether this is a recipe step, a `lym batten` verb, or orch guidance alone.

Waits until the adopt redesign (`adopt-redesign`) has settled how run state moves.
