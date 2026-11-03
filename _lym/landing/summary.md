# lym ide spawn opens a beam's workspace file

`lym ide spawn <beam>` now regenerates and opens the beam's workspace file `<hub>/<RepoName>-<beam>.code-workspace`, as the prime opens the hub workspace.
Before, only the prime got a workspace file; a beam opened its bare worktree folder, so the reed `code` button in the `beam` group never reached the beam's parent, `Bolt` or its pairs.
A task slug still opens its bare folder.

## Design decisions

- A slug counts as a beam when its pair's origin record carries the beam mark, the same mark the workspace regeneration writes a beam's file by.
  `isBeam` in `internal/ideengine/spawn.go` reads it through `fabricengine.ReadOriginFor`.
- An unreadable or absent origin record degrades to the bare folder with a `Warn` log, like a failure to resolve the prime's name.
  Once a slug is a beam, every error from the regeneration is returned, never degraded.
- A beam spawn or seed regenerates the hub workspace file and every beam's file through `fabricengine.RegenerateHubWorkspace`, then launches the beam's own file.
  The `.vscode/` config is written first, so the beam's file splices the settings just written.
- `lym ide seed` follows the same path for a beam and launches nothing.

## Tests and docs

- `spawn_hub_integration_test.go` covers a beam seed (title, no launch), a beam spawn (file launched, beam first folder, window command run once), and the degrade for a missing records lock directory and for an absent origin record.
- `docs/overview.md` and the package comments in `internal/ideengine` and `internal/idecli` describe the beam path.

## To check by hand

- Click the reed `code` button on a real beam and confirm VS Code opens `<RepoName>-<beam>.code-workspace` with the beam, its parent, `Bolt` and its pairs.
- The new integration steps were not run in this description step; run `lym gate test ./internal/ideengine`.
