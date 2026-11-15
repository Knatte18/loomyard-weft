# Move a running task by hand-back and a fresh run, not a re-parent verb

`move-task-by-handback` · Note · feature, batten, loom, board

## Seen

The study `docs/research/loom/reparent-running-task-2026-11-12.md` (commit `5e41ee2b`) finds that a verb moving a running task to another parent is feasible but wide:
two durable stores written by two modules with no atomic change, a narrow board write past the run lock,
an ancestry check on the new parent, and a cross-module tapestry run of its own.

## Idea (operator, 2026-11-15)

Moving a running task is rare, and a verb for it is clumsy for what it buys.
Instead, end the run and start a new one with more information:

1. The run's live writer writes everything it has learned to a hand-back file in its run records:
   what is done, the decisions taken, what is left, and where its commits are.
   The entry is held by the run and takes no body edit, so the file is the only place it can write.
2. The run is cancelled with `lym batten cancel`, which sets the entry back to `draft` and frees it.
   The archive tag keeps the run records, the hand-back and the commits, so the new run can read them.
3. The orch writes the hand-back into the entry's body, or into a new task under the new parent, and releases it.

What the design must settle:

- Whether the new run re-uses the old commits (a cherry-pick, which brings back the study's ancestry question) or only reads them.
- Whether this is a recipe step, a `lym batten` verb, or orch guidance alone.

The adopt redesign (`adopt-redesign`) has landed; the builds of how run state moves are the members of the project `adopt-build`, still open (`run-fsm-tracked-records`, `push-before-act-takeover`).
Waits until those have landed.
