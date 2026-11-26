# Show each beam's and the root's autonomy and PR setting on the board

Until now, the only way to learn whether autonomy is on for a beam, for the root or hub-wide, and which `pull_request` mode its members land by, was to read `beam.yaml` files.
The rendered board now shows both settings, read the way the runs read them.

## What the board files show

- The README header states the hub-wide default of autonomy and of the pull-request setting, with the legend of the mode lines.
- The section of the root (`P0`) and of each open beam carries a mode line, in the README and in each project file. Example:
  `Autonomy _on_ (P0) · PR setting **merge** · Parent branch `main` · Beam PR [#12](…)`.
  - A **bold** value is that project's own setting, set in its own worktree's files.
  - An _italic_ value is inherited, with the place it comes from in brackets: a project's handle, `P0` for the prime, or `hub-wide`.
- A beam's line also names its parent branch, from its origin record, and its recorded pull request as a link, from its landing record.
- A setting that the root and every beam take from the hub-wide default stands in the README header alone.
  The section lines leave it out.

## How the values are read

- `dobbyengine.LookupWithSource` is the per-task lookup that runs already use, unchanged in what it answers.
  It now also returns which anchor of the chain set the value, or `HubWideSource` when the hub's default layer or the template decides.
  `lookupTaskSetting`, behind `LookupPullRequest`, `LookupLandNotice` and `LookupAutonomy`, now calls it.
- `hubgeom.TaskSettingChain` records each anchor's worktree name in the new `Names` field, `""` for the prime.
- `hubgeom.BeamModes` is the reader.
  It takes the board directory and the open beams' slugs.
  It reads the hub-wide settings, then the root's chain, then each beam's chain from the beam's worktree up.
  It finds the prime from the board directory alone, without spawning git.
  The Bolt checkout is a worktree of the records repository, and that repository's main worktree is the prime's records sibling, which `lymcwd.ResolveWithin` maps to the prime.
- `boardengine` cannot import `hubgeom`, which imports it.
  The reader is therefore installed process-wide with `boardengine.InstallBeamModeReader` in the lym root's pre-run, next to the Bolt replayers.
  `Config.Outputs` carries it in the new `Outputs.BeamModes` field, so every `RenderToDisk` path uses it.
  Those paths are the board write, the owner write and the Bolt replay.
- A test binary installs no reader, so existing renders and their tests are unchanged.
  `Render`, which reads no hub, renders no mode line.
  A failed read renders the board with no mode line and logs a warning.
  A beam whose own read fails is left out with a warning.

## Design decisions

- The added decision to mark inherited values apart from local overrides is met with bold and italic plus a source in brackets, not colour.
  Neither GitHub nor VS Code's markdown preview renders colour from plain markdown.
- The beam's pull request comes from its landing record, with no call to GitHub, so a render makes no network call.
  The link stands while the record names a pull request.
- `lym board list --text` is unchanged; the entry left it optional.

## Open risks

- A render reads the beam settings of this machine.
  A beam whose worktree is absent here is left out of its chain, as it is for a run here, so its parent's value shows, and it has no pull-request link.
  On a hub that spans machines, a render on a machine that does not own the beam can therefore write different mode lines than the owner's render, and the README changes between the two.
- The mode lines change only when a board write renders the files again.
  A `beam.yaml` edit shows at the next board write.

## Tests

- `TestLookupTaskSettings` now asserts, on each existing row and for each key, the anchor `LookupWithSource` names.
  The rows cover the task's own file, a parent, the prime and the hub-wide default.
- `TestBeamModes` (tier 1, `internal/hubgeom`) drives the reader over a hubforge hub.
  It covers a value the root sets, which the beam inherits, a value the beam sets itself, a hub-wide value, and the parent branch and pull request of the beam.
- `TestRenderToDisk_ShowsTheInstalledBeamModes` (tier 0, `internal/boardengine`) installs a fake reader.
  It pins which value a mode line marks as the project's own, which it marks inherited and from where.
  It also pins that a setting every place takes from the hub-wide default stands in the README header alone.

## Merge danger

**Door:** two-way
**Blast radius:** board
