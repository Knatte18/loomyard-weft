# Hub workspace file lists every pair, named by project handle

The hub workspace file `<hub>/<repo>.code-workspace` now lists every pair of the hub, including the pairs under beams at any depth.
Before, it listed only pairs whose parent is the prime, so a beam's tasks showed up only in the beam's own file.
One VS Code window now shows all of a repository's code, and the machine restart launcher, which opens only the hub file, relies on that.

A pair whose board entry belongs to a project is named `P1: adopt-worker-record` in that file, the project's handle leading the slug.
A pair with no project, or whose project the board holds with no handle, keeps its slug alone.
The handle is read from the board at each regeneration, so a regeneration after a board change brings the names current.
Beam workspace files are unchanged.

## Design decisions

- `fabricengine` gets a general seam, `PairFolderNamer` with `InstallPairFolderNamer`, mapping pair slug to folder name for a hub path.
  Fabric knows nothing of the board; with no namer installed every pair is named by its slug.
  This keeps fabric from importing `boardengine`.
- `boardengine.WorkspaceFolderNames` is the namer lym installs in the root `PersistentPreRunE` in `cmd/lym/main.go`.
  An unreadable board is logged at Warn and yields no names, so regeneration never fails over a folder name.
- `regenerateHubWorkspace` reads the names before it takes the hub workspace lock.
  The namer reads the board under locks of its own, and the hub workspace lock is a leaf lock.
- Beam files name their pairs by slug only; the handle prefix is a hub-file feature.
- `docs/overview.md` and the `fabricengine` and `boardengine` package docs describe the new behaviour.

## Tests

- `TestWorkspaceFolderNames` covers nested projects, entries without a project, a project the board lacks, and an unreadable or missing board.
- `TestRegenerateHubWorkspace` gains a naming case and updated expectations: the hub file holds beam members and sub-beam pairs.
- `TestPersistentPreRunE_InstallsSeams` requires the root pre-run to call `fabricengine.InstallPairFolderNamer`.

## To check by hand

- Run `lym ide spawn` on the prime in a hub with a beam holding tasks, and confirm the hub window shows those tasks named `<P#>: <slug>`.
  The root pre-run install is skipped under `testing.Testing()`, so no automated test runs the board-backed namer end to end.
