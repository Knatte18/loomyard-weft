# Rename lyx to lym (LoomYard Mill): binary, LYXHUB suffix, LYX_ variables and _lyx/.lyx directories

`rename-binary-lym` · Note · enhancement, stencil, shuttle

Imported from [issue #670](https://github.com/Knatte18/loomyard/issues/670).

`lyx` collides with LyX, the LaTeX editor, which ships as the `lyx` package in Ubuntu/Debian and installs `/usr/bin/lyx`.
On a machine with both, whichever comes first on PATH wins: a shell where `~/go/bin` is not yet on PATH starts the LaTeX editor instead of LoomYard.
LyX is common among researchers, so the clash is likely for other users too.

## Name

The new name is `lym`, for LoomYard Mill, after LoomYard's forerunner millhouse and its Mill plugin (operator, 2026-10-26).
It clashes with no installable command in apt, Homebrew, winget, Scoop, Chocolatey or AUR; the note `binary-name-candidates` holds the check.

## Priority (operator, 2026-10-26)

The rename lands before LoomYard is taken into use on other repositories, since renaming hubs and records already in use elsewhere is far more work.

## Scope

- The binary: the `cmd/lyx` build target and `binaryName()` in `tools/deploy`, docs and README, stencils and skills in the `ly` plugin that invoke `lyx`, and any scripts or launchers that call it.
- The hub directory suffix `-LYXHUB`, such as `loomyard-LYXHUB`, which becomes `-LYMHUB` (operator, 2026-10-26).
- The `LYX_*` environment variables, which become `LYM_*`, and the `_lyx/` and `.lyx/` directories, which become `_lym/` and `.lym/` (operator, 2026-10-26).
  Existing hubs need a migration of their records and machine-local state.
- Dobby's misnamed `markers` directory goes in the same change, since it holds no marker files (operator, 2026-10-26): `dobby.log` moves up beside `state.json` and `dobby.lock` as `.lym/beam/dobby.log`, and the board-tidy reports move to `.lym/beam/tidy/`, named by their timestamp alone, such as `20261026T151739Z.md`.
  `MarkerDirName` in `internal/dobbyengine/geometry.go` goes with it.

## Done mechanically (operator, 2026-10-26)

The rename is done by a script, never by hand edits file by file.
Agents are allowed scripted edits since `scripted-edits-allowed` landed: the shell guard no longer denies `sed -i`, `perl -i` or a redirect into a project file.

## Runs alone (operator, 2026-10-22)

The rename runs alone, with no other run in flight, since it touches nearly every file; it is promoted and started only then.
