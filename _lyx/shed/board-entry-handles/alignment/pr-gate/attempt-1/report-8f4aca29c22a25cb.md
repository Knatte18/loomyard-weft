---
verdict: APPROVED
---

## Measure

The board entry `board-entry-handles` as written, since the decision record carries no `Added after Discussion` entry.
The diff of pull request 824 at `1a826b20a87fd0a9c53785b152c94a6236b53f6a` against `main`, whose merge base `0d953f140` is `main`'s head.

## Operator decisions against the diff

1. Tasks carry no priority: `migrateEntries` drops a task's priority on load, `normalizeTask` refuses a task with one on every upsert, merge and replay path, and `Store.Promote` clears it while `stampDates` keeps the promoted note's dates.
2. Stored handles, one counter per kind or type: `Task.Handle` is stored as `handle` in board.json, and `assignHandles` gives the lowest free number of the entry's letter, keeps a held handle, frees a removed entry's number, moves a promoted note to the lowest free `T`, and assigns existing entries once in id order on load.
   The letters come from the hub's `board.yaml` types, as the orch recommendation suggests, and `LoadConfig` refuses labels that would share a letter, which that derivation needs.
3. Every board verb that takes a slug or id takes a handle: `get`, `remove`, `promote`, `rename`, `set-status`, `set-deps` and merge's `set_status` take a `handle` key or a bare handle such as `lyx board get B4`.
4. README: the handle replaces the row number, notes stand one table per type sorted by handle with an `H`/`L` priority column, tasks keep Running, Ready, the layers and Independent, each sorted by handle, and age and revised share one `Age<br>Revised` column.
5. `lyx board list --text` and `find --text` print the handle first.

The replay change, where an added entry takes a handle free on origin's board, keeps point 2's stable lowest-free handle across machines and is no work beyond the entry.
The handle is left out of `scopeChanged`, so a run-held entry still takes its handle.

## Open risks carried over

The decision record's two open risks, a hub whose type letters collide being refused until renamed and a handle seen locally before a push changing once on replay, name no unrecorded parent correction.

## Overlap

- Peer `run-process-ownership` (row 6): its decision record reads the local board's owner verdict through `boardengine`, but it changes no handle, priority, selector or README behaviour and lists no changed paths.
  No overlap.
- Peer `finalize-dirty-names-paths` (row 5): its decision record touches no board behaviour and lists no changed paths.
  No overlap.
- Parent commits: none listed, and `main` has not moved past the merge base.

No overlap applies, so no landing order is needed; for the record, this run at row 21 is further along than both peers.

## Superseded

None: the related board notes, such as `board-viewer`, ask for work this run does not do.
