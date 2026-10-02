# Card 4 — board-json-store

**Edit:**
- `internal/boardengine/store.go`
- `internal/boardengine/board.go`
- `internal/boardengine/render.go`
- `internal/boardcli/cli.go`
- `internal/boardengine/boardtest/bench_test.go`
- `internal/boardengine/boardtest/concurrency_test.go`
- `internal/boardengine/store_test.go`
- `internal/boardengine/board_test.go`
- `internal/boardengine/render_test.go`
- `internal/boardcli/notes_test.go`
- `internal/boardcli/promotenote_test.go`
- `internal/boardcli/cli_test.go`
- `internal/loomcli/landingdeps_integration_test.go`

**Uses:**
- `internal/boardengine/legacy.go`
- `internal/state#ReadJSON`
- `internal/state#WriteJSON`
- `internal/fabricengine#BoardWriteLockFile`

**Intent:**
Switch the board to one store file, `board.json`, migrating from the legacy files on load and folding pre-upgrade done marks, and collapse the two-store scaffolding (Decisions 1, 8, 9, 11, 13).

`store.go`:
- `Store` is built over the board directory, not one file, and holds the entries and `legacy_done`.
- The on-disk shape is `{"version": 1, "entries": [...], "legacy_done": [...]}`, with `legacy_done` omitted when empty; a `version` other than 1 refuses the load, naming the version found.
- `Load`: when `board.json` exists, read it under `board.json.swaplock`; otherwise, when a legacy file exists, read both legacy files (each under its own `.swaplock`, as the old binary wrote them) and migrate them through card 3's functions; with neither, the store is empty.
  Whenever a legacy file exists, run card 3's fold over the loaded entries.
  Legacy existence is checked before any legacy read, so a board without legacy files never gains a legacy `.swaplock`.
  `Load` never writes.
- `Save` writes `board.json` only; no code path writes `tasks.json` or `notes.json`.

`board.go`:
- `boardCriticalSection` becomes the one locked write helper — `board.lock`, load, mutate, save `board.json`, render, spawn sync — over a single `*Store`.
  It keeps its name because `internal/fabricengine`'s `BoardWriteLockFile` doc cites it.
  `writeOp`, `storeTarget`, and `checkCrossStoreSlugAvailable`/`checkCrossStoreSlugsAvailable` are deleted.
- Every `*Note*` method (`UpsertNote`, `SetNoteStatus`, `RemoveNote`, `MergeNotes`, `SetNoteDeps`, `UpsertNotesBatch`, `GetNote`, `ListNotesBrief`, `ListNotesFull`) and `taskToUpsertFields` are deleted.
- `PromoteNote(idOrSlug)` runs card 2's `Store.Promote` to tier 1 under the lock, and on an entry already at tier 1 returns it unchanged and succeeds without a write.
- `HealthCheck` passes when the board directory holds a readable `board.json` or a readable legacy file.
- The read methods (`GetTask`, `ListTasksBrief`, `ListTasksFull`) load the one store, so a read on an unmigrated board migrates in memory and persists nothing.
- The package doc describes one store with legacy migration and fold, and its closing roadmap paragraph says the board is now the roadmap, with `manifest/` retirement left to task `manifest-retire`.

`render.go`: `Render` and `RenderToDisk` take one entry list; `renderManifestSection` is deleted, so every entry renders through the existing tasks section until card 6 rewrites the shape.

`internal/boardcli/cli.go`: each `notes` subcommand calls the top-level `Board` method of the same verb, and `promote-note` calls `PromoteNote`; card 7 restructures them into aliases.

`boardtest`: `bench_test.go`'s `seedWiki` writes a `board.json` and `BenchmarkRender` calls the one-list `Render`;
`concurrency_test.go`'s comments name `board.json`.

Tests:
- `store_test.go`: load from `board.json`; nil `depends_on` normalization and a corrupt file over `board.json`; an unknown `version` refused.
- `board_test.go`: a board holding only legacy files reads as migrated and a read leaves the directory byte-identical;
  the first write creates `board.json`, leaves both legacy files in place, and creates no `tasks.json` or `notes.json` on a board that had none;
  a slug in both legacy files refuses the write and leaves every file byte-identical;
  a pre-upgrade-shaped `SetStatus` — the legacy `tasks.json` rewritten with the entry `done` under `board.lock` through `state.WriteJSON` — followed by any new-binary write leaves that entry `done` in `board.json`;
  `PromoteNote` moves a tier-3 entry to tier 1 and succeeds unchanged on a tier-1 entry;
  `HealthCheck` passes on `board.json` alone and on a legacy file alone.
  The cross-store slug and two-store promote tests are deleted.
- `render_test.go`: callers move to the one-list signature; `TestRenderManifestSection` is deleted.
- `internal/boardcli`: `notes_test.go` asserts a `notes upsert` lands in `board.json` and `notes list` returns every entry;
  `promotenote_test.go` asserts the promoted entry's tier is 1 and a second call succeeds unchanged;
  `cli_test.go`'s board-path test asserts `board.json` in the derived board dir.
- `internal/loomcli/landingdeps_integration_test.go`: a hub board seeded with a legacy `tasks.json` holding the pair's slug has that entry `done` in `board.json` after `MarkTaskDone`.

**ImpactSummary:** Storage and facade rewrite inside boardengine; external callers keep their method names, `boardcli` repoints its notes call sites, and the loomcli landing test gains a migrated-entry case.
