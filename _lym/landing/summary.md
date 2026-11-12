# Study what a re-parent of a running task has to write

Adds one report, `docs/research/loom/reparent-running-task-2026-11-12.md`, and its line in `docs/research/README.md`.
No code changes, no verb is written, and nothing was run on a live task.

## What the report finds

- A task's parent is two durable values: the board entry's `project` and the pair's `_lym/fabric/origin.json`.
  The batten run record holds no parent of its own.
  The loom status file and the merge record carry copies, and the parent lock, a beam's members and the dobby's view are derived.
- `hubgeom.ParentOf` reads the origin record first, so the two durable values can disagree, and the origin record wins for landing, ownership and blockers.
- A claimed entry refuses a `project` edit through `checkRunLocks`.
  A pause holds no writer of either durable value, so a verb needs its own pause caller and a narrow board write.
- The two stores cannot change atomically.
  Writing the origin record first makes a crash between the writes harmless, and the verb must be idempotent.
- Git: a re-parent is safe only when the new parent holds the commits the task started from.
  Merging the new parent at Parent-Align does not remove the old beam's commits, so the report recommends refusing.
- Beam-Await reads its blockers from the board and the origin records at the land request, so the old beam needs no edit.
  The new parent must pass the same preconditions as a fork from it.
- The report ends with the verb's refusals, write order, tests and size: recipe `tapestry`, profile `solid`.

## Design decisions

- The report is filed under `loom` in `docs/research/`, since the study spans landing, runs and the board.
- Claims the study did not verify are marked in the report and listed in its last section, rather than guessed.
- `_lym/drift/expected.yaml` does not exist for this task, and no captured behavior changes.
