# Board entries get short handles (T4, B2, E7); tasks lose priority

Every board entry now carries a short, stable handle that names it in any board verb, so the operator can say `T4` or `B2` instead of a long slug.
README row numbers shifted whenever an entry was added or removed, and the stored `id` is shared across all kinds and runs into the hundreds, so neither named an entry reliably.
A task's priority only distracted, since a note that became a task is prioritized by being one, and the README note tables were short of width.

## What changes

- **Handles.** An entry's handle is a letter and a number, stored as `handle` in `board.json`.
  A task gets `T`; a note gets the upper-cased first letter of its type label, which gives the loomyard hub's `bug`, `enhancement`, `feature` and `audit` exactly `B`, `E`, `F` and `A`.
  A new entry gets the lowest free number of its letter, a number freed by a removed entry is given out again, and adding or removing other entries never changes a handle.
  A promoted note drops its note handle and takes the lowest free `T`.
  A `board.json` without handles gets them in id order on load, and the next write persists them.
- **Lookup.** `get`, `remove`, `promote`, `rename`, `set-status`, `set-deps` and merge's `set_status` take `handle` beside `slug` and `id`, exactly one of the three.
  A bare handle argument also works, case-insensitively, as in `lyx board get B4`.
  A verb that needs further keys refuses a bare handle as a missing required field.
  `depends_on`, merge's `remove_slugs` and intake import's `into` stay slug fields.
- **Tasks carry no priority.**
  `promote` clears the note's priority and keeps its dates.
  An upsert or merge that leaves a task with a priority is refused, naming `promote` and `"priority":"normal"` as the ways forward.
  A task stored with a priority loses it on load.
- **README.**
  The handle replaces the row number in the first column.
  Notes stand in one table per type, sorted by handle, with a `P` column holding `H` for high, `L` for low and empty for normal.
  Tasks keep their groups, each sorted by handle, with no priority split.
  Age and revised share one column, age above revised (`Age<br>Revised`).
  Done stays one table, tasks first, each kind by handle.
  `RenderOrder` follows the same order, so `list` and `find` agree with the README.
- **`list --text` and `find --text`** print the handle as the first column.

The module docs of `boardengine` and `boardcli` and the `ly:board` skill describe the new behavior.

## Design decisions

- **One assigner.**
  A single function assigns handles over the whole entry list, on load, in the write critical section after the mutation, and in the replay before its save.
  An entry keeps a stored handle whose letter is its own and that no lower-id entry holds; every other entry, in id order, gets the lowest free number.
  A note whose type label is no longer configured keeps its stored handle.
- **Letters come from hub config.**
  No letter list is hard-coded: the letter is the first letter of the type label in `board.yaml`.
  `LoadConfig` therefore refuses type labels that do not begin with a letter, that share a first letter, or that begin with `t`, which `T` reserves for tasks.
  The loomyard hub's and the template's types pass; a hub with a colliding `board.yaml` fails every board verb until a label is renamed.
- **A handle is not scope.**
  The run lock and the revised date ignore a handle change, so a run-held entry still takes one.
  `handle` is not an upsert payload key, and a payload carrying it is refused because the board assigns it.
- **Replay.**
  An entry added by a replayed change takes a handle free on origin's board, never the local one, and the replay does not merge `handle` as a field.
  Two machines adding entries concurrently each assign the same lowest-free handle locally, so a handle read from a local listing before the push can change once.
- **Older binaries.** `handle` is a declared field, and an older binary keeps it as an unknown key, as the tolerant-records invariant requires.

## To check by hand

- Run `lyx board list --text` and `lyx board get B4` on a real hub board, and confirm the README renders the per-type note tables and the merged `Age<br>Revised` column as intended.
- Confirm the loomyard hub's `board.yaml` types load without a letter-collision refusal.
