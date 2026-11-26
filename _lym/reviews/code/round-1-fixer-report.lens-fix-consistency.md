```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of this round's change set (commits 728b9da4b, 75c1e98ba, f90f456b9 and the `summary.md` edits).

1. **MEDIUM**: the legend and the header-only rule in `internal/boardengine/beammodes.go#modeHeader` no longer hold now that F1 leaves an absent beam out.
   - The README says "A setting that the root and every beam take from the hub-wide default is stated here alone."
   - `everyPlaceHubWide` looks only at the root and the beams in `b.modes.Beams`, so an absent beam is not counted. That beam may set its own value on its owner machine, yet the setting is still dropped from every line and the README states it in the header alone.
   - Before the fix, an absent beam was in the map. Now its section has no mode line, and a reader of that legend infers that it takes the hub-wide default.
   - Neither the legend nor `internal/boardengine/doc.go` ("A setting that the root and every beam take from the hub-wide default stands in the README's header alone") says that "every beam" means the beams read on this machine.

2. **LOW**: `internal/boardengine/doc.go`, README section, was not updated for F1.
   - It still says "The section and file header of the root and of each open beam carry a mode line."
   - The only exception it names is "A failed read renders the board with no mode line".
   - A beam whose worktree is not on this machine now carries no line. `hubgeom/doc.go` and the `BeamModes.Beams` comment say so, but this twin does not.

3. **LOW**: `pattern/PATTERN-config-layers.md:129` says "The board files show the root's and each beam's value, and the worktree that set it, through the same lookup". That is no longer true for a beam without a worktree on the rendering machine.

4. **LOW**: `_lym/landing/summary.md` was only partly brought in line.
   - Under "What the board files show", it still says "The section of the root (`P0`) and of each open beam carries a mode line". This disagrees with the rewritten Open risks bullet.
   - The "Tests" section does not mention what the fix added. For `TestBeamModes`, that is the absent beam being left out. For `TestRenderToDisk_ShowsTheInstalledBeamModes`, it is the inherited-from-a-beam and unknown-slug sources, the project files' lines, the done and non-beam filter and the failed read.

Tool calls:
1. One Bash call: read `pattern/PATTERN-config-layers.md` around line 130, diffed `cmd/lym/main.go`, `boardengine/config.go` and `hubgeom/tasksetting.go`, and grepped md and yaml files for mode-line mentions. My context showed none of these. Nothing in the three diffs disagrees with the fix.
