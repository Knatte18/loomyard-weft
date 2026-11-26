# Show each beam's and the root's autonomy and PR setting on the board

The board files did not show whether autonomy is on for a beam, for the root or hub-wide,
so the operator had to read `beam.yaml` files to know which halts and pull requests would reach them.

## What changes

The README's header now states the hub-wide default of autonomy and of the pull-request setting, with a legend.
The README section and the project file header of the root and of each open beam carry a mode line:

- autonomy, on or off, and the `pull_request` setting;
- for a beam, its parent branch and the pull request its landing record names, as a link.

A **bold** value is the project's own setting.
An _italic_ value is inherited, with its source in brackets: another project's handle, `P0`, a worktree slug or `hub-wide`.

## Design decisions

- `hubgeom.BeamModes` reads the values through `TaskSettingChain` and the per-task lookup the runs use,
  from each beam's worktree up.
  `dobbyengine.LookupWithSource` is that lookup, now exported and also answering which worktree set the value;
  `TaskSettingChain` gains `Names` for the same reason.
- `boardengine` does not read the hub itself.
  The `lym` root installs the reader with `boardengine.InstallBeamModeReader`.
  With no reader, as in test binaries and in `boardengine.Render`, no mode line is written.
- A beam whose worktree is not on this machine gets no mode line, rather than its parent's values shown as inherited.
  The legend says its settings may differ.
- A failed read never fails the render.
  A beam that cannot be read loses its line with a warning,
  and a failure for the prime, root or hub-wide values drops the header and every mode line.
- The header always states the hub-wide default.
  A setting that the root and every beam read take from that default is left off the mode lines.
- Own versus inherited is marked with bold and italic, not colour.
- Rendering reads no GitHub state, so the `Beam PR` link may point to a closed or merged pull request.
  The legend says so.
- `lym board list --text` is unchanged.

## To check

- A beam whose worktree is absent on the rendering machine must not show its parent's values
  (`readBeamMode` in `internal/hubgeom/beammodes.go`).
  The code leaves such a beam without a mode line.
  Because the board files are shared, a render on that machine also removes the line the owner machine wrote.
- The boardengine tests use a stand-in reader and the hubgeom test calls `BeamModes` directly.
  Render a real hub's board with the built `lym` and compare the mode lines with the `beam.yaml` files.

## Merge danger

**Door:** two-way
**Blast radius:** board
