# board.yaml is hub-wide, read and written at the board dir

All seven batches reached `status: done`, each committed by its fork.

- **01 configreg-hub-wide** (`51d85f244`): `configreg.Module.HubWide`, set for `fabric` and `board`.
- **02 bolt-commit-written** (`e802552dd`): `Bolt.CommitWritten` writes and commits named paths under the board write lock.
- **03 hub-wide-reconcile** (`c6d0221b4`): `ReconcileHubWideAt` replaces the fabric-only reconcile and seeds `board.yaml` from the prime, a legacy copy or the template.
  `fabric reconcile` commits the result in `_board`.
- **04 board-readers-hub-file** (`88e61057b`): `lyx board`, loom's landing board and `lyx ide` menu load the hub `board.yaml`.
  `LoadConfig` refuses with a `lyx fabric reconcile` way forward when the file is absent.
- **05 retire-worktree-copy** (`d9bb0cb77`): `ReconcileAll` skips hub-wide modules by the registry field and retires subsumed per-worktree copies, reporting `retired` and `divergent`.
- **06 configengine-staged-edit** (`d920230e9`): `EditPath` and `StagingFile` for editing an untracked staging copy.
- **07 config-cli-hub-wide** (`609291c4b`): `lyx config` edits, sets, prints and menus hub-wide modules at the board dir and commits in `_board`.
  `PATTERN.md` gains `PATTERN-hub-wide-config`.

## Deviations

Batch 07 changed two files outside its declared file-ops: `cmd/lyx/configstrictness_test.go` and `pattern/PATTERN-config-strictness.md`.
The fork reports that the existing strictness guard failed without both, since `configcli` is now a strict `configengine.Load` caller.

## Verify gate fixes

The gate's first attempt failed on six `TestBattenIntegration_*` tests in `internal/battencli`.
Each stopped with `board config <hub>/_lyx/config/board.yaml not initialized; run "lyx fabric reconcile"`, because the `seedBoardTask` fixture helper loaded `board.yaml` from the worktree anchor, not the hub board dir that batch 04's reader now uses.
The fixer fork's reply says it changed `seedBoardTask` in `internal/battencli/lifecycle_integration_test.go` to load from `h.BoardDir()`, which covers all six tests, and that `go test -tags integration ./internal/battencli` passes.
Its fix is commit `03a6887f3` ("fix: battencli fixtures load board config from the hub board dir").
