# lym ide spawn on a beam opens the beam's workspace file

`lym ide spawn <beam>`, which the reed `code` button in the `beam` group runs, now regenerates the workspace files and opens the beam's own `<hub>/<RepoName>-<beam>.code-workspace` (the beam, its parent, `Bolt` and the beam's pairs), as `lym ide spawn <prime>` opens the hub workspace.
A task slug still opens its bare folder.

## What changed

- `internal/ideengine/spawn.go`: `seed` treats a slug whose origin record carries the beam mark as a beam.
  For a beam it regenerates through `fabricengine.RegenerateHubWorkspace`, after writing the beam's `.vscode/` config, and sets the launch path to `fabricengine.BeamWorkspacePath`; the window command runs in the hub directory, as for the prime.
  `lym ide seed <beam>` regenerates the files too and launches nothing; `lym ide menu` gets the new behaviour through `Spawn`.
- Package and CLI docs and the ide entry of `docs/overview.md` name the beam case.
- `internal/ideengine`: a new scenario step, `beam opens the beam workspace`, marks a pair a beam and asserts the seed writes the beam's file with its workspace title and launches nothing, and the spawn launches that file with the beam as its first folder and runs the window command once.

## Design decisions

- Beam-ness is read from the pair's origin record, the same mark the regeneration writes a beam's file by, rather than the board's beam flag, so the file opened is always one the regeneration just wrote.
- An unreadable origin record is logged at Warn and degrades to the bare folder, matching the existing degrade for an unresolvable prime; once a slug is known to be a beam, a regeneration error is returned, as on the prime path.
- `lym ide open` is unchanged: it opens a task's folder and writes nothing.
