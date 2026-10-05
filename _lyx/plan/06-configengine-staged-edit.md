# Card 6 — configengine: edit a file at a told path, and name its staging file

**Create:**
- `plan:internal/configengine#EditPath` -> `func EditPath(path, template string, edit EditorFunc) error`
- `plan:internal/configengine#StagingFile` -> `func StagingFile(baseDir, module string) string`

**Edit:**
- `internal/configengine#Edit`
- `internal/configengine/edit.go`
- `internal/configengine/config.go`
- `internal/configengine/edit_test.go`
- `internal/configengine/config_test.go`
- `docs/shared-libs/configengine.md`

**Intent:**
Card 7 runs the interactive editor for a hub-wide module on a staging copy, so the board write lock is never held while an operator's editor is open (see Shared Decisions).
This card adds the two primitives that needs, with no behaviour change for existing callers.

`EditPath` is `Edit`'s body after its `FindBaseDir` check: it scaffolds `path` from `template` when absent (creating its parent dir), runs the editor, loops on a YAML parse error and returns `ErrAborted` on an unchanged unparseable save, removing a file it scaffolded on any abort.
`Edit` becomes `FindBaseDir` plus `EditPath(ConfigFile(baseDir, module), template, edit)`, so its contract and its tests are unchanged.
`scaffoldIfMissing`'s `configDir` argument is derived from the path's parent inside `EditPath`.

`StagingFile` returns `filepath.Join(baseDir, lyxdirs.DotLyxDirName, configDirName, module+".yaml")`, the ephemeral `.lyx` counterpart of `ConfigFile` at the mirrored subpath (`PATTERN-durable-vs-ephemeral-state`, `PATTERN-lyxdirs-single-declarer`), so a staged edit is never tracked.

Docs: `docs/shared-libs/configengine.md` documents `EditPath` and `StagingFile` beside `Edit` and `ConfigFile`.
It also corrects its directory tree, which places `board.yaml` and `fabric.yaml` under a worktree's `_lyx/config/`, to show them under `<hub>/_board/_lyx/config/`, with a one-line note that `configreg` marks them hub-wide.
Its two generic `ConfigFile` examples that name `_lyx/config/board.yaml` use a per-worktree module such as `loom.yaml` instead.

Coverage:
- `edit_test.go`: one test that `EditPath` on a path whose parent dir does not exist scaffolds the template there, applies the editor's write and returns nil.
  The existing `Edit` tests already pin the loop and abort paths through the shared body.
- `config_test.go`: one assertion that `StagingFile(base, "board")` lies under `base`'s `.lyx` dir and differs from `ConfigFile(base, "board")` only in that segment.

**ImpactSummary:** `Edit` is refactored onto the new `EditPath` with an unchanged contract; `StagingFile` is additive.

**Commit:** 6: configengine: EditPath and StagingFile for staged config edits
