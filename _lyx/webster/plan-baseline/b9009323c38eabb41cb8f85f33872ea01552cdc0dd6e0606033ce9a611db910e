# Card 6 — board-draft-migration

**Edit:**
- `internal/boardengine#Store`
- `internal/boardengine#storeFile`
- `internal/boardengine#loadedStoreFile`
- `internal/boardengine#loadedStoreFile.UnmarshalJSON`
- `internal/boardengine#Store.Load`
- `internal/boardengine#Store.Save`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#FabricReplay.Apply`
- `internal/boardengine#turnIntoProject`
- `internal/boardengine#tableRow`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine_test#TestRenderReadmeGolden`
- `internal/boardengine/doc.go`

**Create:**
- `internal/boardengine#migrateDraftStatus`
- `internal/boardengine_test#TestDraftMigration`

**Uses:**
- `internal/boardengine#StatusDraft`
- `internal/boardengine#IsEmptyStatus`

**Intent:**
Migrate an existing board once, keep the mark across replays and older binaries, and show a draft task in the README (D5).
`board.json` gains the top-level key `draft_status` (`true` once migrated): `storeFile` and `loadedStoreFile` carry it as a known field, `Store` holds it, `Load` reads it and `Save` writes it, and an older binary keeps it as an unknown key (`PATTERN-tolerant-records`); the store's on-disk version stays 1, since `boardengine`'s store accepts one version.
`migrateDraftStatus` runs in `boardCriticalSection` before `applyDraftRule`, only while the key is absent: it marks every open task (not done) whose status is empty `draft`, sets the key, and reports whether it marked anything; the write it rides on carries both in the same save, and a board holding the key is never migrated again.
`FabricReplay.Apply` carries the commit's `draft_status` onto origin's store as it carries `prime_owner` (a commit that sets it sets it on origin; neither side ever clears it), so a replayed migration commit cannot leave origin unmarked and re-migrate released tasks on its next write.
`turnIntoProject` clears a `draft` status, since a project carries no draft (promote's claim clear writes `draft` before it turns the entry into a project).
`tableRow` marks a draft task's title cell with `draft` before the bold title in the Ready and layer tables; a draft task keeps its Ready or layer section and its After column (`ComputeLayers` is unchanged, as `draft` is no run status).
The package doc's README and migration sections state the mark, the key and the once-only migration, and the accepted bound: a task an older binary creates after the migration has no status and starts without a release until every machine runs the new binary.
Tests: `TestDraftMigration` (tier0, external, parallel) rows: a board without the key whose next write marks an open empty-status task (nil and empty string) `draft`, leaves a done task and a `running · X` task alone and writes `draft_status`; a board holding the key whose released task stays nil on the next write.
`TestFabricReplay` gains a row: a commit that set `draft_status` replays onto an origin board without it and the replayed board holds it.
`TestSetOwners` gains a row: a `ToProject` write over a `draft` task leaves the project with a nil status.
`TestRenderReadmeGolden`'s golden fixture gains a draft task in Ready rendered with the `draft` mark.

**ImpactSummary:** A known top-level key on board.json read and written by the store, carried by the replay; render changes only a draft task's row.
