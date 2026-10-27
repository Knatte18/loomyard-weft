# Rename lyx to lym (LoomYard Mill): binary, LYXHUB suffix, LYX_ variables and _lyx/.lyx directories

`rename-binary-lym` · Task · enhancement, stencil, shuttle · running · PR-Review

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
- The `LYX_*` environment variables, which become `LYM_*`, and the `_lyx/` and `.lyx/` directory names in the code, which become `_lym/` and `.lym/` (operator, 2026-10-26).
- Dobby's misnamed `markers` directory goes in the same change, since it holds no marker files (operator, 2026-10-26): `dobby.log` moves up beside `state.json` and `dobby.lock` as `.lym/beam/dobby.log`, and the board-tidy reports move to `.lym/beam/tidy/`, named by their timestamp alone, such as `20261026T151739Z.md`.
  `MarkerDirName` in `internal/dobbyengine/geometry.go` goes with it.
- No migration code: the binary knows only the new names, and existing hubs are cloned afresh (see Cutover).

## Done by a script (operator, 2026-10-27)

The run is a darn whose writer writes one rename script and runs it, opening hardly any file itself: the diff is the script's output, plus the few hand fixes the gate asks for.
The script stays in the run's `.scratch/`, never committed.
It changes text and tracked file and directory names in the repository only.
It never moves the worktree's own `_lyx` link or the records under it, and never touches the weft side: the run's own state lives there and the running binary is still `lyx`.

## Cutover (operator, after the landing)

The landing deploys nothing: `deploy_branches` in the landing config is `[]` (set 2026-10-27), so the old hub keeps the old `lyx` until the cutover.
Once the rename has landed, the old hub runs nothing more:

1. Clone loomyard into a plain directory beside the hub and run `update-plugins.sh` there, which installs `lym` and the renamed `ly` plugin.
2. Rename `_lyx/` to `_lym/` in one commit on Bolt and on `main-weft`, and push both.
3. Clone the hub afresh with `lym fabric clone`, as `loomyard-LYMHUB`, and start it with `lym beam start`.
4. Set `deploy_branches` back to `["main"]`.

The `ly` plugin is global, so the other hubs (quarry, cs, and OSL-1046's) move to `lym` at the same time and are cloned afresh the same way.
Machine-local state is not carried over; the orch note is copied by hand.

## Runs alone (operator, 2026-10-22)

The rename runs alone, with no other run in flight, since it touches nearly every file.
