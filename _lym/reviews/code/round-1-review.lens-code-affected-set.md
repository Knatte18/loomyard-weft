```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set I found and whether it is covered:

1. **`buttonContext` and the new `pairIsBeam` in `buttons.go`.**
   - Reader: `reedengine/buttonrow.go:75` is the only caller of `geom.Buttons()`. It treats any error as "render no buttons" with a Warn, so the new board-read error path degrades safely. Covered.
   - Tests: `hubgeom_test.go` `TestReedGeometry_ButtonContext` is updated, and I found no test that asserts the old error text. The calls in `geometry_integration_test.go:162` and `:202` were not read for the error text, but the board-absent case returns an empty list (`Store.Load`: absent file gives `[]`). Covered.
   - Docs: `hubgeom/doc.go:83-85` is updated. `reedengine/doc.go:69` only mentions `Geometry.Buttons`, with no place rule. Covered.

2. **Place semantics, `ButtonPlaceBeam` versus `ButtonPlaceTask`.**
   - `buttonBeam` (`buttons.go:87`) and `buttonrow.go:95-109` branch on place alone, so a beam read from the board flag gets the beam group and its own worktree as the button dir, with no further code change needed.
   - Another place classifier: `ideengine/spawn.go:128` `isBeam` and `spawn.go:338` also classify a beam. Their reads differ: `isBeam` reads the origin mark only, and `spawn.go:338` reads the board with the beam flag, requires an open project, and excludes done entries. This is outside the diff and no finding is raised. See finding F-A below for the done-entry difference.

3. **PATTERN entries.**
   - `PATTERN-owner-machine` and `PATTERN-fabric-boundary` list no rule that `hubgeom` buttons may not read the board. `hubgeom` already imports `boardengine` in `parentof.go`, so no new import edge. No pattern change is declared or needed. Covered.

**Findings**

- **F-A, LOW, consistency.**
  - Location: `internal/hubgeom/buttons.go:72-74`.
  - Summary: `pairIsBeam` accepts a done beam project (`Kind == project && Beam`) with no done check, while the sibling readers `boardengine.ParentBeam` (`parentbeam.go:32`, `!isDone`) and `ideengine/spawn.go:338` skip done entries.
  - Effect: a landed or done beam whose board entry stays flagged will keep the beam group. If a slug is reused by a later task pair, it would read as a beam.
  - Evidence: `parentbeam.go:32`, `spawn.go:338`. Impact is low, because the origin mark would already say beam for a finished beam.

No other affected-set member was left wrong. Commit messages carry no `Pattern-Departure:` trailer, and none is needed.

Tool calls:
1. grep for beam-mark and button-context prose across `.go`, `.md` and `.yaml`: to find docs and callers that describe the place rule, which the manifest lacked.
2. grep for `hubgeom` and `boardengine` in three PATTERN backgrounds, and for `.Buttons(` callers: to check pattern constraints and the set of readers.
