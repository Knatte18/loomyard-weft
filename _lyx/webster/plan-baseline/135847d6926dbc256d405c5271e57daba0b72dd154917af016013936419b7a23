# Card 7 — tokencount reports a measured Merriam fixed context

**Create:**
- `tools/tokencount#directiveStencilPath`
- `tools/tokencount#merriamTexts`
- `tools/tokencount#FixedRow`
- `tools/tokencount#FixedContext`
- `tools/tokencount#Calibration.addFixed`
- `tools/tokencount#FixedContext.write`

**Edit:**
- `tools/tokencount#readRunStartBase`
- `tools/tokencount#Calibration`
- `tools/tokencount#Calibrate`
- `tools/tokencount#Calibration.WriteMarkdown`
- `internal/websterengine#merriamFixedContext`
- `tools/tokencount/main.go`
- `tools/tokencount/calibrate_test.go`
- `tools/tokencount/calibrate_integration_test.go`

**Uses:**
- `internal/pattern#FillDirective`
- `tools/tokencount#MerriamStart`

**Intent:**
D4's report: the calibration gains a Merriam fixed-context section, so the operator can re-measure `merriamFixedContext` from real runs, and the start fit's reconstructed base becomes the texts `MerriamBase` counts.

`merriamTexts` reconstructs, at the run's base commit, exactly the texts `websterengine.MerriamBase` counts but `00-overview.md`:
- `CLAUDE.md`, adding nothing when absent at the base commit;
- the Master template (`masterTemplatePath`), its absence a reason;
- the orchestrator PATTERN directive, rendered by `pattern.FillDirective` from the base commit's `directiveStencilPath` and `PATTERN.md`, with `directiveStencilPath` as the stencil name, never the raw `PATTERN.md`: an absent `PATTERN.md` adds nothing, as `pattern.Directive` renders nothing for it, and a `PATTERN.md` with the stencil absent is a reason.
A run with no base, or a base commit not in the repository, has a reason; a failed read is an error.
`CLAUDE.local.md`, which `MerriamBase` also counts, is untracked and cannot be reconstructed: the derivation assumes the run's worktree had none, as a lyx-created worktree has none, and `merriamTexts`' comment says so.

`readRunStartBase` keeps its plan lookup and builds its base from `merriamTexts` plus the plan's `00-overview.md`, so the start fit prices the base `MerriamBase` computes and both sections reconstruct one set of texts.

The section:
- `FixedRow` is one run's measurement: `Run`, `Session`, `Measured` (the session's `StartContext`), `Lines` (the summed line count of `merriamTexts`, counted as `websterengine.MerriamBaseOf` counts), and `Fixed`, Measured minus Lines times the profile's `context_per_line`.
- `FixedContext` holds the rows, its own skips (`CalibrationSkip`), the median and spread of `Fixed` over the rows (the 25th and 75th percentiles, as `fitOfRatios` computes them), and `Current`, the constant read as `websterengine.MerriamBaseOf().Fixed`, so the tool needs no new export.
- `addFixed` measures each run's first Merriam session by `Started` from `run.Merriams`; a session with a `NoStart` reason, or a run whose `merriamTexts` gives a reason, is a skip with that reason.
  Every later Merriam session of the run (a resume, a refresh, a re-launch) is a skip with the reason that it starts with prior conversation or a compaction summary in context.
- `Calibration` gains the `Fixed` field, `Calibrate` fills it per run, and `WriteMarkdown` writes a `Merriam fixed context` subsection: one table row per run with the measured start, line count and derived fixed figure, the median and spread, the current constant beside them, and the section's skips with their reasons.

`merriamFixedContext`'s comment names the calibration's Merriam fixed-context section as its source and says the figure covers the system prompt, tools and the skills loaded at launch; the value stays as it is, for the operator to update from the report.
`main.go`'s package comment names the new section.

Tests:
- `calibrate_test.go`: the base-file helpers add the orchestrator directive stencil at the base commit, and the known base context counts the rendered directive in place of the raw `PATTERN.md`;
  in `TestCalibrate`, a fixture run with a Webster session measured at its prompt Read writes the section with that start, a line count equal to the lines of the base commit's `CLAUDE.md`, Master template and the directive rendered from its `PATTERN.md` (so the directive wrapper's lines are subtracted), the derived fixed figure, the median and the current constant;
  a second Merriam session of the same run is listed among the section's skips with its reason, not measured;
  a run with `PATTERN.md` but no directive stencil at its base is a skip in both sections.
- `calibrate_integration_test.go`: the git-backed fixture commits the directive stencil, so `TestCalibrateGitBackedMatchesInMemory` still matches the in-memory calibration, the new section included.

**ImpactSummary:** the start fit's reconstructed base now counts the rendered directive instead of the raw `PATTERN.md`, so reported estimates shift by the directive wrapper's lines; one new report section; the constant's value is unchanged.

**Commit:** 7: tokencount: Merriam fixed-context calibration over the texts MerriamBase counts
