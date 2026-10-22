# Board entries get short handles; only notes keep a priority

Every board entry now carries a short, stable handle such as `T4`, `B2` or `E7`, so the operator can name an entry without typing its slug.
Only notes carry a priority, and the README tables open with the handle and are narrower.

## Handles

- `Task.Handle` is stored in board.json as `handle`.
  A task's letter is `T`, and a note's is the upper-cased first letter of its type label, so the hub's `board.yaml` `types` decide the note letters.
  The loomyard hub's `audit`, `bug`, `enhancement` and `feature` give `A`, `B`, `E` and `F`.
- `assignHandles` (new `internal/boardengine/handles.go`) runs over the whole entry list on `Store.Load`, in `boardCriticalSection` after the mutation, and in `FabricReplay.Apply` before its save.
  An entry keeps a stored handle of its own letter that no lower-id entry holds, and every other entry, in id order, takes the lowest free number of its letter.
  So a handle stays while its entry lives, a freed number is given out again, and a promote or a type-label change moves the entry to the lowest free number of its new letter.
  A board.json written before this change gets handles in id order on its first load, and the next write persists them.
- A replayed added entry drops its local handle and takes one free on origin's board, so two machines' concurrent adds never collide.
- A handle is not scope: the run lock and the revised date ignore it, and the replay does not merge it as a field.
  An upsert payload carrying `handle` is refused, naming that the board assigns it.
- `LoadConfig` refuses type labels that do not begin with a letter, that share a first letter, or that begin with `t`, naming the label to rename.

## Lookup by handle

- The engine takes a `boardengine.Handle` selector beside a slug string and a numeric id, through one `selects` matcher that `GetTask`, `RemoveTask`, `SetStatus`, `SetDeps`, `Promote` and `Rename` now share.
  The letter matches case-insensitively.
- In the CLI, `get`, `remove`, `promote`, `rename`, `set-status`, `set-deps` and merge's `set_status` take exactly one of `slug`, `id` or `handle`.
  `payloadKeys.exclusive` became groups of any size for this, and the `Use` tokens read `{slug|id|handle…}`.
- A payload argument that is a bare handle reads as `{"handle":…}`, so `lyx board get B4` works.
  A verb that needs further keys refuses a bare handle as missing them.
- `list --text` and `find --text` print the handle as the first column.

## Priority on notes only

- `normalizeTask` refuses a task carrying a priority, so an upsert, batch or merge that gives a task one is refused, naming `"priority":"normal"` and `lyx board promote` as the ways forward.
- `Promote` clears the note's priority and keeps its dates.
- A task stored with a priority loses it on load, in `migrateEntries`, so the run lock and the dates see no change and a run-held such task stays writable.
  The next write drops the key.

## README

- The handle replaces the row number in the `#` column.
- Each group, Running, Ready, a layer, Independent, a Notes type or Done, is one table, with no priority split and no `####` headings.
- `RenderOrder` sorts tasks by layer and then by handle, sorts notes by handle, and puts Done's tasks before its notes, so `list` and `find` follow the same order.
- Notes tables add a `P` column after the handle: `H` for high, `L` for low, blank for normal.
- Age and revised share one `Age<br>Revised` column, the age above the revised.

## Design decisions

- **Letters come from the type label's first letter, not a new config key.**
  The operator's letters fall out of the loomyard types as they stand, so no hub config edit is needed at landing.
  A hub whose types collide gets a refusal at config load rather than ambiguous handles.
- **Handles are assigned on load as well as on write.**
  Reads show handles before the first write after deployment, and the order is deterministic, so the first write persists exactly what the reads showed.
- **Task priority is dropped on load rather than by a migration write.**
  A one-off write would be refused by the run lock on any run-held task that still carries a priority.
- **Only the entry selector takes a handle.**
  `depends_on`, merge's `remove_slugs` and intake's `into` stay slugs, since they are stored or compared slug values and a handle changes on promote.
- **The `--text` handle column is left out when no listed entry has a handle**, matching how the listing's other optional columns behave; on a real board every entry has one.

## Docs

- `internal/boardengine/doc.go` gains a Handles section and rewrites the Priority and README sections.
- `internal/boardcli/doc.go` and the verbs' `--help` describe the `handle` key and the bare argument.
- The `ly:board` skill (`plugins/ly/skills/board/SKILL.md`) gains a Handles section, and its priority text now says notes only.
  It reaches agents at the next `update-plugins.sh`.

## Tests

- New `handles_test.go` covers lowest-free assignment, stability across adds and removes, reuse of a freed number, reassignment on promote and on a type change, lookup by handle in every selector-taking write, the refused `handle` payload key, and assignment on load (a stored handle kept, a duplicate reassigned) persisted by the next write.
- Existing tests gain rows: letter-collision refusals in `TestLoadConfig_LabelsRefused`, a dropped task priority in `TestLoad`, a held note's priority change in `TestRunLock`, a handle free on origin in `TestFabricReplay`, and the handle key, bare argument and malformed-handle refusals in the CLI lookup contract.
- `TestPriorityRoundTrip`, `TestRenderOrder`, the README golden and the compact-listing tests are rewritten for the new rules.
