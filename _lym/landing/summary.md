# Board README: drop the mode legend, show both settings on each mode line

The top of the board's README held a legend explaining how the mode lines read.
It also left autonomy or the PR setting off every mode line whenever the root and every beam took it from the hub-wide default,
so an operator had to look up at the header to learn a project's setting.

## What changes

- `modeHeader` in `internal/boardengine/beammodes.go` now writes only the line opening "Hub-wide default:",
  after the render time and the prime owner.
  The legend lines on bold and italic values, on left-out settings, on unread beams and on the beam PR link are gone.
- `modeLines` always shows autonomy and the PR setting for the root (P0) and for every beam the reader answered,
  bold when the project sets it itself and italic with its source otherwise, `(hub-wide)` included.
  The `everyPlaceHubWide` check that dropped a setting from every line is removed.
  Parent branch and beam PR parts follow as before.
- An open beam the reader left out, such as one whose worktree is on another machine, now carries the mode line
  `Autonomy and PR setting not read on this machine`, which replaces the legend's sentence explaining why such a beam had none.
  A done beam the reader left out still has no mode line, since the reader is asked for open beams only.
- A failed read still renders the board with no header line and no mode lines.
- The package doc in `internal/boardengine/doc.go` describes the new header and mode lines.

## Design decisions

- The unread-beam line is a judgment call on the wish that every beam carries a mode line:
  with the legend gone, a beam with no line at all would leave the operator guessing.
  The render has no values for such a beam, so the line says the settings were not read instead of guessing them.

## Tests

`TestRenderToDisk_ShowsTheInstalledBeamModes` now pins that a setting every place takes from the hub-wide default still shows on each mode line,
and gains an open beam the reader leaves out, pinning its unread line in the README and in its project file.

## Merge danger

**Door:** two-way
**Blast radius:** board

The run `pr-modes-no-agent`, now in Finalize, adds a line about the PR modes to the same legend in `modeHeader`.
Whichever lands second gets a textual conflict there; it resolves by dropping that legend line along with the rest.
