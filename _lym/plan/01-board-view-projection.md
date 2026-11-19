# Card 1 — board-view-projection

**Create:**
- `internal/boardengine#BoardDataFile`
- `internal/boardengine#BoardView`
- `internal/boardengine#ViewProject`
- `internal/boardengine#ViewEntry`
- `internal/boardengine#EntryRevision`
- `internal/boardengine#BoardRevision`
- `internal/boardengine#buildView`
- `internal/boardengine#boardDataScript`
- `internal/boardengine#renderFiles`
- `internal/boardengine#boardView.unmetDependencies`
- `internal/boardengine#migrateLoaded`
- `internal/boardengine#Board.View`
- `internal/boardengine_test#TestRenderBoardData`

**Edit:**
- `internal/boardengine#Render`
- `internal/boardengine#RenderToDisk`
- `internal/boardengine#boardView.afterCell`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#Outputs`
- `internal/boardengine#Config.Outputs`
- `internal/boardengine_test#TestOutputs`
- `internal/boardengine_test#TestRenderToDiskManifestCleanup`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#withMissingRecordsUndiscussed`
- `internal/boardengine#layerSection`
- `internal/boardengine#notesTypeOrder`
- `internal/boardengine#typeHeading`
- `internal/boardengine#boardView.childProjects`
- `internal/boardengine#Board.loadStore`
- `internal/boardengine#DecisionFileName`
- `internal/boardengine#RootSubject`

**Intent:**
Add the board's view projection (D1) and its first render output, `board-data.js`, plus the revision functions D8 and D10 read.

- `BoardDataFile` names the file; `Render` returns it beside the README.
  Its content is `window.BOARD = ` + the JSON of the `BoardView` + `;` + a newline, built by `boardDataScript` with `json.Marshal`.
  Go's default HTML escaping stays on, so a `<`, `>` or `&` in any field is written as a JSON unicode escape, a backslash and `u` followed by `003c`, `003e` or `0026`, and a body holding `</script>` cannot end the script.
- `Render` keeps its signature: it calls `renderFiles(tasks, tasks, ...)`.
  `RenderToDisk` calls `renderFiles(tasks, withMissingRecordsUndiscussed(boardPath, tasks), ...)`.
  `renderFiles` renders every existing file from `rendered`, exactly as `Render` did, and adds `BoardDataFile` from `buildView(stored, rendered, ...)`.
  So each entry's revision is taken over the stored entry, while its decision-file name follows the copy that clears `Discussed` for a missing record.
  RenderToDisk's manifest lists `board-data.js` through the existing manifest write, so stale-file removal covers it.
- `BoardView` JSON keys: `rendered` (RFC 3339 UTC render time), `prime_owner`, `types` (configured type labels in order), `areas` (configured area labels in order), `sections` (README section names in README order), `projects`, `entries`.
  `ViewProject` keys: `handle`, `slug`, `title`, `brief`, `labels`, `parent` (parent project's handle), `design_file`.
  `ViewEntry` keys: `handle`, `slug`, `title`, `kind`, `types`, `areas`, `status` (empty for none), `project` (its project's handle), `section`, `waits_on`, `brief`, `body`, `owner`, `recipe`, `created`, `revised`, `design_file`, `decision_file`, `revision`.
  The design and decision file names are empty when the file does not exist: a design file exists when the body is non-empty (`out.DesignPrefix + slug + ".md"`), a decision file when the rendered copy is discussed (`DecisionFileName`).
- `Outputs` gains `Labels []string`, the configured area labels in board.yaml order, which `Config.Outputs` fills; the type/area split of an entry's labels reads `Outputs.Types`, so a type label never sits among an entry's areas.
  `TestOutputs` compares the whole struct, so its "all fields" row configures a plain label and wants it in `Labels`, and its other row wants the empty `Labels` `Config.Outputs` gives.
- `buildView` builds the projection from the same `boardView` the README renders from (`RenderOrder`, `newBoardView`), so the viewer recomputes nothing:
  - projects: the root `P0` first (slug `RootSubject`, no parent, no design file), then every open project in README order by walking `childProjects` from the root, each carrying the handle of the project it was found under (`P0` for a top-level one) as `parent`;
  - entries: every task and note, done ones included, in render order;
    `project` is the handle of its project, `P0` for none; an entry whose project is not an open project carries that project's handle;
  - `section` reads the README's own section values: `runningSection.name` for the Running layer, `readySection.name` for Ready, `layerSection(layer).name` for a letter, Independent or Draft layer, `typeHeading` or `otherNotesHeading` for an open note by `noteType`, and `doneSection.name` for a done entry;
  - `sections` lists the distinct section names in the README's order: Running, Ready, the dependency layers in render order, Independent, Draft, then the note sections in `notesTypeOrder` with Other last, then Done;
  - `waits_on` holds the handles of the entry's unmet dependencies, the slug for one the board holds no handle for; `unmetDependencies` returns those slugs in `depends_on` order, and `afterCell` now formats its cell from it, so the README and the projection share the rule and the README output is unchanged;
  - `owner` is the effective owner the README's owner cells read (`ownerOf`), `recipe` is `out.recipeOf` for a task and empty for a note.
- `EntryRevision` is the lowercase hex SHA-256 of `json.Marshal(t)`, the entry's own encoding, unknown keys included; it covers every stored field, status included.
- `BoardRevision` is the hex SHA-256 of `board.json`'s bytes under `boardPath`, and `""` with no error when the file is absent.
- `migrateLoaded` runs `migrateDraftStatus` then `migrateDiscussedStatus` on a loaded store; `boardCriticalSection` calls it in place of the two calls, so the write section and the view migrate through one step.
- `Board.View` loads the store through `loadStore`, runs `migrateLoaded` on it without saving, and returns `buildView(store.Tasks(), withMissingRecordsUndiscussed(b.boardPath, store.Tasks()), store.primeOwner, b.out, b.now())`.
  It is the projection the server (card 4) serves.
- `doc.go`'s `# README` section gains a paragraph naming `board-data.js`, its `window.BOARD` shape and the stored-entry revision.
- `render.go`'s file header names `board-data.js` among Render's outputs.

`TestRenderBoardData`, a new tier0 test in `internal/boardengine_test`, runs over one board of projects, a sub-project, a done project with an open sub-project and a task of its own, tasks in several layers, a draft, notes of two types, a done entry, a type and an area label, an entry with a body and a discussed entry, with a prime owner and a configured default recipe:
- strips the `window.BOARD = ` prefix and the `;\n` suffix of `Render`'s `board-data.js` and decodes the JSON;
- asserts each task's and note's `section`, `project` and `waits_on` match the README's placement of that entry, read from the same `Render` call's README, and that each open project lists its parent's handle;
- asserts the open sub-project of the done project names `P0` as its parent, as the README nests it under the root, and the done project's task carries the done project's handle;
- asserts `sections` equals the README's section headings in their README order, `types` and `areas` equal the configured lists, `prime_owner` the owner, and `rendered` the render time;
- asserts the entry with a body names its design file and an entry without one names none, a project's `design_file` likewise, a task's `recipe` is the default recipe and a note's is empty, and `owner`, `created` and `revised` carry the stored values;
- asserts a body holding `</script>` and `<b>` appears with no literal `</script>` or `<b>` in the file;
- asserts a type label never appears among an entry's `areas`;
- asserts each entry's `revision` equals `EntryRevision` of the stored entry;
- through `RenderToDisk` on a board whose discussed entry has no decision file: the written `board-data.js` names no `decision_file` for it, and its `revision` is `EntryRevision` of the stored, still-discussed entry;
- `BoardRevision` gives `""` for an absent `board.json` and the SHA-256 of the bytes for a present one;
- `Board.View` over a `board.json` the load migrations change (no `draft_status` key and a task with an empty status) gives that entry the revision of the migrated entry, and leaves `board.json`'s bytes unchanged.
`TestRenderToDiskManifestCleanup`'s seeded-manifest row asserts `board-data.js` is written and listed in the manifest.
`TestRenderReadmeGolden` keeps pinning that the README is unchanged by the `afterCell` refactor.

**ImpactSummary:** boardengine-internal: Render's signature is unchanged, so its callers (RenderToDisk, renderOwnerWrite, tests) are untouched; RenderToDisk and the owner-set write now also write and list board-data.js; Outputs gains one field filled by Config.Outputs.

**Commit:** 1: board: render board-data.js, the board's view projection
