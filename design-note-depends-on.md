# A note carries depends_on, shown on the board and kept when promoted

`note-depends-on` · Task · enhancement, board · running · Drift-Baseline

Depends on: [`per-task-settings-fixups`](design-per-task-settings-fixups.md)

## Seen

Only a task carries `depends_on` (`lym board set-deps` and `upsert` refuse it on a note; `promote` takes it at the promotion).
A note that must wait for a running task, such as `harden-fabric-undriven-cases` waiting for `harden-fabric-single-repo-verbs`, shows nothing of that on the board,
so the operator keeps asking whether it can start.

## Wanted

Operator, 2026-11-23:

- `upsert` and `set-deps` accept `depends_on` on a note.
- `lym board list --text` and the README show a note's dependencies as they show a task's.
- `promote` keeps a note's `depends_on`; one given in the promote payload replaces it.
- `prune` strips a done entry from a note's `depends_on` as from a task's.
- The `ly:board` and `ly:board-policy` skills say a note may carry `depends_on`.
