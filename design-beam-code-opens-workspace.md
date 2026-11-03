# lym ide spawn on a beam opens the beam workspace

`beam-code-opens-workspace` · Task · enhancement, ide · running · Code-Burler-Review

## Seen (operator, 2026-11-03)

The reed `code` button in the `beam` group runs `lym ide spawn {slug}`.
`ideengine.Spawn` opens the hub workspace file only for the prime; for any other slug, a beam included, it opens the worktree folder.
lym already writes a workspace file per beam, `fabricengine.BeamWorkspacePath` (`<hub>/<RepoName>-<beam>.code-workspace`), listing the beam, its parent, Bolt and the beam's pairs.

## Expected

`lym ide spawn <beam>` regenerates and opens the beam's workspace file, as the prime opens the hub workspace; a task slug still opens its folder.
