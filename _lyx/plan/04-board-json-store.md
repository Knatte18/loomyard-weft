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
- `Load`: when `board.json` exists, read it under `board.json.swaplock`; otherwise, when a legacy file exists, read the legacy files that exist (each under its own `.swaplock`, as the old binary wrote them) and migrate them through card 3's functions; with neither, the store is empty.
  Whenever a legacy file exists, run card 3's fold over the loaded entries.
  Each file's existence is checked before it is read, so a read never creates a `.swaplock` for a file that is absent.
  `Load` never writes.
- `Save` writes `board.json` only; no code path writes `tasks.json` or `notes.json`.

`board.go`:
- `boardCriticalSection` becomes the one locked write helper — `board.lock`, load, mutate, save `board.json`, render, spawn sync — over a single `*Store`.
  It keeps its name because `internal/fabricengine`'s `BoardWriteLockFile` doc cites it.
  It takes an optional after-save step that runs under the lock after the save and before the render; every method passes none until card 5's `RetireLegacy`.
  `writeOp`, `storeTarget`, `checkCrossStoreSlugAvailable`/`checkCrossStoreSlugsAvailable`, and the `tasksFile`/`notesFile` constants are deleted, so card 3's legacy constants are the only declarers of the legacy file names.
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
- `board_test.go`: a board holding only legacy files reads as migrated, and the read leaves `tasks.json` and `notes.json` byte-identical, `board.json` absent, and no README or design doc written;
  these data-file comparisons never include `*.swaplock` files, which a locked read creates and which are gitignored scratch;
  the first write creates `board.json`, leaves both legacy files in place, and creates no `tasks.json` or `notes.json` on a board that had none;
  a slug in both legacy files refuses a read (`GetTask` and `ListTasksBrief`) and a write, each naming the slug, and leaves every data file byte-identical;
  a read over a `board.json` beside a legacy `tasks.json` carrying a new `done` mark returns that entry `done` and leaves `board.json` byte-identical;
  a pre-upgrade-shaped `SetStatus` — the legacy `tasks.json` rewritten with the entry `done` under `board.lock` through `state.WriteJSON` — followed by any new-binary write leaves that entry `done` in `board.json`;
  `PromoteNote` moves a tier-3 entry to tier 1 and succeeds unchanged on a tier-1 entry;
  `HealthCheck` passes on `board.json` alone and on a legacy file alone.
  The cross-store slug and two-store promote tests are deleted.
- `render_test.go`: callers move to the one-list signature; `TestRenderManifestSection` is deleted.
- `internal/boardcli`: `notes_test.go` asserts a `notes upsert` lands in `board.json` and `notes list` returns every entry;
  `promotenote_test.go` asserts the promoted entry's tier is 1 and a second call succeeds unchanged;
  `cli_test.go`'s board-path test asserts `board.json` in the derived board dir.
- `internal/loomcli/landingdeps_integration_test.go`: a hub board seeded with a legacy `tasks.json` holding the pair's slug has that entry `done` in `board.json` after `MarkTaskDone`.

**ImpactSummary:** The deleted `*Note*` methods and `taskToUpsertFields` and the one-list `Render`/`RenderToDisk` have no caller outside `internal/boardengine` and `internal/boardcli`, so only `boardcli`'s notes call sites move; external callers (`loomcli`, `battencli`, `ideengine`) keep their method names, `ideengine`'s tests that seed a legacy `tasks.json` keep passing through migration on load, and the loomcli landing test gains a migrated-entry case.
