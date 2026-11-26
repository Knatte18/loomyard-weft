```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering everything outside the touched lines that each change in the diff forces.

**Per change, the set found, covered or not**

1. `dobbyengine.LookupWithSource` is newly exported, and `lookupTaskSetting` now delegates to it.
   - `internal/dobbyengine/doc.go:139` is covered (updated in the diff).
   - `pattern/PATTERN-config-layers.md` per-task section is covered (bullet added).
   - `PATTERN.md:16` and `pattern/PATTERN-config-layers.md:3,119` still say "only `dobbyengine`'s one lookup". They stay true, because `lookupTaskSetting` is now a thin wrapper over the same single implementation.
   - `internal/dobbyengine/config.go:58` is **not covered**. It still says the per-task keys "are read through LookupPullRequest, LookupLandNotice and LookupAutonomy alone".
   - Tests: `TestLookupTaskSettings` is covered.
2. `dobbyengine.TaskSettingChain.Names` is a new field.
   - The only constructors are `hubgeom.TaskSettingChain`, which fills it, and `BeamModes`'s hub-wide chain at `beammodes.go:41`. That chain has no Anchors, so `LookupWithSource` always answers `HubWideSource` for it and `Names` is never indexed. Covered.
   - No other production literal of the struct exists.
3. `hubgeom.TaskSettingChain` now also appends `Names`. Its callers are `Autonomy`, the dobby/landing deps and `BeamModes`. Behaviour is unchanged for all of them, and the doc comment was updated. Covered.
4. `boardengine.Outputs.BeamModes` is a new field, and `Config.Outputs()` now fills it from the installed reader.
   - The producers are `board.go:46`, `replay.go:589` and `ownerset.go:331,336`. All go through `cfg.Outputs()`, and there is no other `Outputs{}` literal.
   - No `//lym:seam` reflective test walks `Outputs`; the only boardengine seam is on `Task`, at `reportengine/landings_test.go:583`. Covered.
5. `RenderToDisk`, `renderFiles` and `Render`'s behaviour, plus the README header, section and project-file mode lines.
   - `internal/boardengine/doc.go` is covered (updated).
   - `docs/overview.md:478-485` describes the README sections and the `P0` / `project-root.md` rendering. It does not mention mode lines. This is descriptive overview prose, not a contract, so it is not a finding.
   - No stencil reads the README layout. `contracts/stencils/drift/drift-template-judge.md` matched only on generic words.
6. `boardengine.InstallBeamModeReader` is wired in `cmd/lym/main.go`'s pre-run, behind `!testing.Testing()`.
   - Its callers are the pre-run and the boardengine test only.
   - The comment beside the wiring is updated.
   - `PATTERN-cli-cobra` is unaffected. Covered.
7. New `hubgeom` imports (`pawlengine`, `loomengine`, `dotgit`, `boardengine`): `boardengine` and `dotgit` were already imported. My searches found no import-set guard naming hubgeom's imports. Covered.
8. `hubgeom` and `boardengine` doc.go both carry `BeamModes` paragraphs. Covered.
9. Commit messages over the range: none carries a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. None is needed: no test limit was widened and no bug fix was claimed.
10. Declared PATTERN changes: the directive declares none, so there is nothing to check.

**Findings**

- **LOW · consistency** · `internal/dobbyengine/config.go:58`. The `Config` doc comment still says the per-task keys are read "through LookupPullRequest, LookupLandNotice and LookupAutonomy alone". The diff added the exported `LookupWithSource` as a fourth reader and updated the same list in `doc.go:139` but not this one. Evidence: `internal/dobbyengine/config.go:56-58` against `internal/dobbyengine/doc.go:139` and diff `003-internal-dobbyengine.diff` (the `LookupWithSource` export).

**Tool calls**

1. A grep for `TaskSettingChain{` literals, `//lym:seam` on boardengine and hubgeom import guards. My context had no list of the struct's constructors or of the reflective seams.
2. A grep of docs and PATTERN for README-layout descriptions and "one lookup" wording. My context lacked the docs that describe these outputs and rules.
3. Reads of `dobbyengine/config.go:52-62` and the `docs/overview.md` README lines, to confirm the stale list and the overview's wording.
