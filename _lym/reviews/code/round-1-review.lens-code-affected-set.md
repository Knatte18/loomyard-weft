```yaml
lens: code-affected-set
outcome: reported
```

**Lens:** code-affected-set, for the arras family-filing diff. I found two stale statements outside the diff's lines. The code paths I traced are covered.

**Set per change**

- **`RaddleRunEntry` and `RaddleFileDeps` new fields/seams**
  - Constructors found: `internal/loomcli/raddlewiring.go` (updated), `internal/testkit/envkit/envkit.go` (updated), `internal/battencli/wire_seams_test.go` (updated).
  - Covered.
- **Wrapping of `ErrRaddleRepointRefused`**
  - The `File` seam at `internal/loomcli/raddlewiring.go:236-242` returns `board.FileRaddle(filing)` unwrapped, so `errors.Is` reaches `internal/loomshed/raddlefile.go`.
  - Covered.
- **Run-status writers for a run's board entry**
  - The non-test callers are `internal/loomcli/wiring.go:225`, `internal/battencli/commitstatus.go:148`, `internal/loomcli/pawlwiring.go:74` and `internal/loomcli/raddlewiring.go:262`.
  - Loom's hook (`wiring.go:251`) and batten's hook (`commitstatus.go:140`) skip under the mark.
  - `wiring.go:297` only writes a status for a Sketch finish.
  - The pawl write at `pawlwiring.go:74` is for a beam run, which starts only after `ClearMotifRun` has removed the mark.
  - Covered.
- **Filing mark in run-record lists**
  - `ChildRecipeRel` is used only in `internal/battencli/commitstatus.go:89`, so the mark's staging is added at the one site that lists run records.
  - Covered.
- **`ClearMotifRun`'s extension to arras**
  - The `internal/beamcli/doc.go` description is updated.
  - Covered.
- **Docs and prose that still say a raddle run ends its entry done**
  - `docs/overview.md:1302` and `contracts/recipes/raddle-recipe.yaml:4` are not updated. See F1 and F2.

**Findings**

F1 — LOW, consistency — `docs/overview.md:1302`
- The line says "A raddle run ends the entry done in its own Raddle-File, never as a Sketch." That is no longer true for an arras family filing, which turns the entry into a discussed project and never ends it.
- The same section's neighbouring loom doc (`internal/loomcli/doc.go`) was updated for this.
- Fix: add the family exception to that sentence, and say the entry becomes the family's project and no run status write reaches it.

F2 — NIT, consistency — `contracts/recipes/raddle-recipe.yaml:4`
- The comment on Raddle-File says it "ends the run's own entry".
- The row is shared with arras, whose family filing does not end the entry.
- This is lower priority because the comment sits in the raddle recipe, where the one-task and tasks outcomes still end the entry.
- Fix: reword it to "ends the run's own entry, unless a family filing turned it into the family's project" or point it at the arras recipe.

**Not flagged**
- The rubric's declared-PATTERN-changes check is moot: no declared changes were visible in the constraints I read.
- No commit carries a `Pattern-Departure:` trailer, so none needs judging.
- The filing mark's path is built from `lymdirs`/`shedrun` constants, which fits `PATTERN-lym-files`.

**Tool calls**
1. grep for `ChildRecipeRel`/`ChildRecipeFile(` callers: the manifest had no list of run-record staging sites.
2. grep for `SetRunStatus(`/`SetBoardStatus` outside `boardengine`: needed every writer of the run's entry status.
3. Read of `internal/loomcli/raddlewiring.go` lines 190-260: needed to see whether the `File` seam wraps errors.
4. grep for `Raddle-File`/`EndRaddleEntry` in md, yaml and tmpl files: needed the docs and recipe files that describe the ending.
5. grep in `docs/overview.md`, `raddle-proposal-spec.md` and the recipe yaml files for "ends the run's entry"-style phrases: needed to find stale statements.
6. sed of `docs/overview.md` lines 1298-1306 plus a grep for arras mentions: needed the context of the stale sentence.
