# A note carries depends_on, shown on the board and kept when promoted

A note that must wait for another entry, such as a running task, can now say so on the board.
Before this change `upsert` and `set-deps` refused `depends_on` on a note, so the wait lived only in the operator's head.

## What changes

- **Writes.** `validateWrite` no longer refuses a note with `depends_on`, so `upsert`, `upsert-batch`, `merge`, `set-deps`, a demotion of a task that has dependencies, the web page's writes and the Bolt replay all accept it.
  The other dependency rules stand: a dependency must exist, must not be isolated, must not be a note, and must not close a cycle.
  An entry still cannot become a note while another entry depends on it.
- **Promote.** `promoteTo` already kept the entry's fields, so a promoted note keeps its `depends_on`; a `depends_on` in the promote payload replaces it.
  The help text and the doc comment now say so.
- **Prune.** `Prune` already stripped removed slugs from every surviving entry, so a note's `depends_on` loses its done entries too; a test now pins it.
- **README.** A note section's table gains an After column, holding the handles of the unmet dependencies as a task's does, when any note in that section waits on one (`boardView.noteTable`).
- **`list` and `list --text`.** `BriefTask` gains `pending`, the dependencies not yet met, computed by `Dependencies.Pending` for every entry.
  `RenderCompact` adds an `after <slugs>` column from it, for tasks and notes alike, left out when no listed entry waits.
- **Siblings.** The sibling notice judges tasks only, since a note never runs: a note waiting on the same entry is never reported as a sibling, and `set-deps` on a note reports none.
  A task's baseline now counts only dependencies it held as a task, so promoting a note that already waits on an entry an open task waits on reports the pair.
- **Docs and skills.** `boardengine` and `boardcli` package docs, the `set-deps`, `upsert`, `promote` and `list` help, and the `ly:board` and `ly:board-policy` skills say a note may carry `depends_on`.

## Design decisions

- A note still cannot be depended on, so a note's `depends_on` names tasks or projects only; the board entry asked for nothing more.
- Making an entry a Sketch still clears its `depends_on` (`withoutRunFields`): the record asked nothing of Sketches, and the Discussion-end scope check compares against that cleared form.
- A note with an unmet dependency now computes a letter layer instead of `Ready` in `list`'s `layer` field.
  README order ignores a note's layer, so only that JSON field shows it.
- The door is one-way because a note written with `depends_on` stays in `board.json` after a revert,
  and a binary without this change refuses any write that touches such a note, the Bolt replay included, until the field is cleared.

## Tests

- `TestKindDependencyRules`: a note takes `depends_on` through upsert, `set-deps` and a demotion; the old refusal subtests are gone.
- `TestPromote`: rows for no payload, a replacing payload and an empty payload, through a new test-only `Store.PromoteWithDeps`.
- `TestPrune`: a note's `depends_on` loses the pruned slug and keeps the live one.
- `TestListTasksBriefLayerAndProposal`: `pending` lists the unmet dependency of a task and of a note and leaves out the done one.
- `TestWriteSiblings`: rows for a promoted note keeping a shared dependency, `set-deps` on a note, and a note that is no sibling.
- The replay row `NoteWithDependsOnIsRefused` becomes `NoteWithDependsOnLands`.

## Merge danger

**Door:** one-way
**Blast radius:** board
