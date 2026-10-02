# Fixer report — plan, round 1

All eight findings are fixed.
No card's targets, type labels or structure changed, and `approved:` in `00-overview.md` was not touched.

## Changed

- **F1** (`01-entry-schema.md`): the tier rule is now stated as one predicate per edge.
  A dependency is exempt when it is `done`, and the dependent's own status never exempts an edge.
  Direction 2 now covers every dependent, `done` or not, unless the incoming entry is itself `done`.
  Added a test: demoting a dependency of a `done` dependent to a higher tier is refused.
- **F2** (`04-board-json-store.md`): `Load` reads only the legacy files that exist and checks each file before reading it, so a read never creates a `.swaplock` for an absent file.
  The read assertion now compares data files (`tasks.json`/`notes.json` byte-identical, `board.json` absent, no README or design doc), and says `*.swaplock` files are excluded.
- **F3** (`04-board-json-store.md`): added two tests.
  The slug-in-both-files refusal now covers a read (`GetTask`, `ListTasksBrief`) as well as a write.
  A read over `board.json` beside a legacy `tasks.json` that carries a new `done` mark returns the entry `done` and leaves `board.json` byte-identical.
- **F4** (`04-board-json-store.md`, `05-board-verbs.md`): `boardCriticalSection` gains an optional after-save step that runs under the lock, after the save and before render.
  `RetireLegacy` drops `legacy_done` in its mutate and deletes the legacy files and swaplocks in that step.
- **F5** (`03-legacy-migration.md`, `04-board-json-store.md`): card 3 names its constants `legacyTasksFile`/`legacyNotesFile`.
  Card 4 deletes `board.go`'s `tasksFile`/`notesFile`, so `legacy.go` is the only declarer.
- **F6** (`04-board-json-store.md`): the ImpactSummary now concludes on the blast radius.
  The deleted `*Note*` methods, `taskToUpsertFields`, and the one-list `Render`/`RenderToDisk` have no callers outside `boardengine`/`boardcli`.
  `ideengine`'s legacy-seeding tests pass through migration on load.
- **F7** (`06-render-roadmap.md`, `10-overview-doc.md`): card 6 says the generated intro and meaning lines are one sentence each and are never hard-wrapped.
  Card 10 says its new prose follows semantic line breaks.
- **F8** (`06-render-roadmap.md`): the ImpactSummary now says the list-order change reaches only `boardcli`'s `list`, the sole production caller of `ListTasksBrief`, and that no stencil reads `lyx board list`.

## Deferred with reason

None.
