# Decision record — board-entry-handles

## Goal

Give every board entry a short, stable handle such as `T4`, `B2` or `E7` that the operator can name an entry by, take priority off tasks, and tighten the README tables around the handle.

## Scope

- `internal/boardengine`: a stored `handle` field on every entry, its assignment on every write and on load, handle lookup in every selector-taking method, task priority removal, README layout, `RenderOrder`, the package doc.
- `internal/boardcli`: a `handle` payload key on every slug|id selector, a bare handle argument, the handle first in `--text`, help text and the package doc.
- Replay (`internal/boardengine/replay.go`): an added entry takes a handle of origin's board, never the local one.
- Out of scope: `depends_on`, merge's `remove_slugs` and intake import's `into`, which stay slug lists or slug fields; `id` stays as it is.

## Decisions

### Handle shape and storage

Decision: an entry's handle is stored in board.json as the string key `handle`, a letter followed by a positive number, such as `B4`.
A task's letter is `T`; a note's letter is the upper-cased first letter of its type label.
Rationale: the type labels are hub config (`board.yaml` `types`), so deriving the letter from the label takes the letter from the hub's config with no new key, and gives the loomyard hub's `bug`, `enhancement`, `feature` and `audit` exactly `B`, `E`, `F` and `A`.

### Letter collisions refused at config load

Decision: `LoadConfig` refuses a `board.yaml` whose type labels do not each begin with a letter, whose two type labels share a first letter, or whose type label begins with `t`, naming the labels and the way forward, renaming a type label.
Rationale: two types on one letter would make handles ambiguous; the refusal is bounded to `types`, and the loomyard hub's and the template's types pass.

### Assignment

Decision: one function assigns handles over the whole entry list, run on load and in the write critical section after the mutation, and in the replay before its save.
An entry keeps a stored handle whose letter is its own and that no entry of lower id already holds; every other entry, taken in id order, gets the lowest number of its letter not in use.
An entry whose letter cannot be decided, a note whose type label is no longer configured or any note on a board with no vocabulary, keeps its stored handle.
Rationale: one assigner keeps the lowest-free, stable and no-history rules in one place; a promote changes the entry's letter and so reassigns it to the lowest free `T`, and a removed entry frees its number; existing entries are assigned once in id order, which is their creation order, and the first write persists them.

### Handle is not scope

Decision: a handle change is never a scope change: the run lock and the revised date ignore it, and the replay does not merge it as a field.
`handle` is not an upsert payload key: a payload carrying it is refused, naming that the board assigns it.
Rationale: the board assigns handles, so a run-held entry must still take one, and two machines' local handles for new entries would otherwise collide.

### Selector

Decision: every board verb that addresses one entry by `slug` or `id` also takes `handle`, exactly one of the three: `get`, `remove`, `promote`, `rename`, `set-status`, `set-deps` and merge's `set_status`.
The argument may also be a bare handle, as in `lyx board get B4`, read case-insensitively; a verb that needs further keys refuses a bare handle as a missing required field.
The engine takes a `boardengine.Handle` selector beside the slug string and the numeric id.
Rationale: the operator decision names `lyx board get B4`; the JSON key keeps the payload grammar for every verb.

### Tasks carry no priority

Decision: a task's priority is dropped on load, so a task stored with one loses it at the next write, and the run lock and dates see no change.
`promote` clears the note's priority and keeps its dates.
A write leaving a task with a priority is refused, naming promote and `"priority":"normal"` as the ways forward.
Rationale: the operator decision; dropping on load keeps a run-held task with a stored priority writable.

### README

Decision: the handle replaces the row number in the first column.
Tasks keep their groups, Running, Ready, the layers and Independent, each one table sorted by handle with no priority split.
Notes stand in one table per type, sorted by handle, with a `P` column after the handle holding `H` for high, `L` for low and nothing for normal.
Age and revised share one column, the age above the revised in one cell joined by `<br>`, the header `Age<br>Revised`.
Done stays one table, tasks first, each kind by handle.
`RenderOrder` follows the same order, so `list` and `find` agree with the README.
Rationale: the operator decision.

### Text listing

Decision: `lyx board list --text` and `find --text` print the handle as the first column.
Rationale: the operator decision.

## Constraints

- PATTERN-tolerant-records: board.json keeps unknown keys; the new `handle` key is a declared field, and an older binary keeps it as an unknown key.
- PATTERN-comment-line-breaks and PATTERN-test-economy: new rows in existing tests where a behavior already has a test.
- CLAUDE.md: `doc.go` of both packages updated in the same commit as the code.

## Auto-mode assumptions

- The README handle column keeps the `#` header.
- The notes' priority column header is `P`.
- Done orders tasks before notes, each by handle.
- A bare-argument handle matches case-insensitively; a stored handle is always upper case.

## Open risks

- A hub whose type labels collide on a first letter or start with `t` refuses every board verb until its `board.yaml` is renamed; the loomyard hub's types pass.
- Two machines adding entries concurrently each assign the same lowest-free handle locally; the replay reassigns the added entry on origin, so a handle named from a local listing before the push can change once.

## Acceptance criteria

- A new task gets `T` with the lowest free number, and a new note the letter of its type with the lowest free number.
- A number freed by a removed entry is given again; other entries' handles never change on an add or remove.
- A promoted note gets the lowest free `T` handle, loses its priority and keeps its dates.
- A board.json without handles gets them in id order on load and persists them on the next write.
- `get`, `remove`, `promote`, `rename`, `set-status`, `set-deps` and merge's `set_status` find an entry by `handle`, and `get B4` by a bare argument.
- An upsert carrying `handle` is refused; an upsert or merge giving a task a priority is refused.
- A task stored with a priority loads without one, and a run-held such task still takes a status write.
- `LoadConfig` refuses type labels sharing a first letter or beginning with `t`.
- The README shows handles in the first column, notes per type sorted by handle with the `P` column, tasks per group sorted by handle, and the merged age/revised column.
- `list --text` prints the handle first.
- A replayed added entry takes a handle free on origin's board.
