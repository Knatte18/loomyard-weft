# A note carries depends_on, shown on the board and kept on promote

A note that must wait for a running task looked ready to start, because the board refused `depends_on` on a note and showed nothing of the wait.
Any entry may now depend on any other: a task, a note or a project.
A cycle is still refused.

## What changes

- `lym board upsert` and `lym board set-deps` accept `depends_on` on a note.
  `validateWrite` in `internal/boardengine/store.go` no longer refuses a note with dependencies, a dependency on a note, or a note that others depend on.
- A dependency on a note is never met while the note exists, whatever its status (`internal/boardengine/dependency.go`).
  It is met once the note has been promoted and landed, or removed, so a task waiting on a note stays blocked.
- `lym board list` entries carry a `pending` field, the dependencies not yet met.
  `lym board list --text` ends a waiting entry's row with `after` and those slugs, in a column that is left out when no entry waits.
  The README's After column shows a note's dependencies as it shows a task's.
- `promote` keeps the note's `depends_on`, and a `depends_on` in the promote payload replaces it.
- Removing an entry strips its slug from every remaining entry's `depends_on`, as `prune` does for a done entry.
  A merge that removes an entry and upserts one of the same slug keeps the dependents' `depends_on`, so they keep waiting.
- `MakeSketch` no longer refuses an entry that others depend on.
  The refusal notice in `internal/loomcli/wiring.go` no longer names dependents.
- A note never runs, so its `depends_on` creates no sibling in the overlap notice (`internal/boardengine/siblings.go`).
- The `raddleparser` check `depends-note` and its entry in `raddle-proposal-spec.md` are removed, since the board no longer refuses what it flagged.
- The `ly:board` and `ly:board-policy` skills and the `boardcli` and `boardengine` package docs say a note may carry `depends_on`.

## Design decisions

- A dependency on a note blocks until the note has become a task and landed, or is removed, rather than until the note is "done".
  A note has no done state, so any other rule would let a task start before the idea it waits on is built.
- Removal strips dependents instead of refusing.
  A refusal would leave the operator to hand-edit every dependent before dropping an entry.

## To check by hand

- Filing a dependency on a note and on a task, then a note-to-note cycle, against a real board: the cycle must be refused.
- `internal/boardengine/store.go`, `validateWrite`: the earlier refusals of a dependency on a note and of a depended-on note are gone, so a note may be written with `depends_on`.
- `internal/boardengine/store.go`, `RemoveTask`: removing an entry leaves no slug of it in any `depends_on`, including through `MergeTasks`.
- `plugins/ly/skills/board/SKILL.md`, `plugins/ly/skills/board-policy/SKILL.md` and the package docs: no sentence still says only a task carries `depends_on` or that a note cannot be depended on.

## Merge danger

**Door:** two-way
**Blast radius:** board
