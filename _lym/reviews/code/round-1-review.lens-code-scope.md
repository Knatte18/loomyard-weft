```yaml
lens: code-scope
outcome: reported
```

Scope lens: I traced each change in the diff to the board entry's goal points and the record's one `Added after Discussion` entry.

**Decisions** (the board entry's `## Goal` points, plus the added entry):
- **G1**: for each beam and for the root (P0), autonomy on or off and where the value comes from (the beam's own `beam.yaml`, the root, or the hub-wide default).
- **G2**: the `pull_request` setting as it resolves for the beam's members.
- **G3**: the beam's parent branch, and the beam's own open pull request as a link when it has one.
- **G4**: state the hub-wide value once at the top when it applies to every beam.
- **G5**: read the values the way runs resolve them, from each beam's worktree up.
- **G6**: optional, `lym board list --text` may show the same.
- **A1** (Added after Discussion, parent): every inheritable value shows whether it is inherited or the beam's own override, with the legend stated once.

**Changes and what they trace to:**
- **C1**: `dobbyengine.LookupWithSource`, `HubWideSource`, `lookupTaskSetting` delegating to it, the source assertions in `TestLookupTaskSettings`, and `dobbyengine/doc.go`. Traces to G1 (where the value comes from), A1 and G5.
- **C2**: the `TaskSettingChain.Names` field and the `hubgeom.TaskSettingChain` change that fills it. Traces to G1 and A1, which need the anchor index mapped to a worktree name.
- **C3**: `hubgeom.BeamModes`, `primeOfBoard`, `readBeamMode`, `readBeamSettings`, `readBeamSetting`, `TestBeamModes` and `hubgeom/doc.go`. Traces to G1, G2, G3 and G5.
- **C4**: the boardengine types, `InstallBeamModeReader`, `Outputs.BeamModes`, `readBeamModes`, `modeHeader`, `modeLines`, `everyPlaceHubWide`, `settingText`, `sourceText`, the README and section wiring in `render.go`, `boardengine/doc.go` and `TestRenderToDisk_ShowsTheInstalledBeamModes`. Traces to G1–G4 and A1.
- **C5**: the mode line in each project file's header (`render.go` `fileHeader`, `boardView.fileHeader`). Traces to nothing: the entry asks for the README's project sections only. See finding 2.
- **C6**: `cmd/lym/main.go` installing `hubgeom.BeamModes`. Forced by C4, which only works with a reader installed.
- **C7**: the `pattern/PATTERN-config-layers.md` bullet. Forced by C1 and C3: the rule that per-task settings are read only through dobbyengine's one lookup needed to name the new reader.

**Which changes build each decision:**
- **G1**: C1, C2, C3, C4.
- **G2**: C3, C4.
- **G3**: C3 and C4 build the parent branch and the link, but the link is not limited to an open pull request (finding 1).
- **G4**: C4 (`modeHeader`, `everyPlaceHubWide`); the header is unconditional (finding 3).
- **G5**: C1, C2, C3.
- **G6**: none. The entry makes it optional, so no finding.
- **A1**: C4, using bold and italic plus the source in brackets; the entry's colour was only an example.

**Findings**

1. **LOW, scope.** Location: `internal/hubgeom/beammodes.go` `readBeamMode` (diff 004, lines ~761–764), and `internal/boardengine/beammodes.go` `modeLines` (diff 002, lines ~173–175).
   - **Summary:** G3 asks for the beam's own *open* pull request. The diff links any pull request the landing record names, so a closed one, or one merged before the beam's Finalize runs, still shows as "Beam PR".
   - **Evidence:**
     - `internal/landingshed/finalize_pr.go` `recordedBeamLanding`: a closed pull request halts and the record stays.
     - `dropBeamRecord` clears the record only after the landing completes (`finalize.go:421`).
     - `_lym/landing/summary.md` says "The link stands while the record names a pull request."
   - **Fix:** limit the link to an open pull request, or have the parent record this narrowing with `lym loom decision add`.

2. **LOW, scope.** Location: `internal/boardengine/render.go` `boardView.fileHeader` (diff 002, line ~456).
   - **Summary:** the diff also puts the mode line into each project file (`project-&lt;slug&gt;.md`, `project-root.md`). The entry scopes the change to "the README's project sections", and no goal point or added entry licenses it.
   - **Evidence:** the board entry's `## Idea` ("In the README's project sections"), and the diff line `lines = append(lines, b.modeLines(project)...)` in `fileHeader`.
   - **Fix:** drop the project-file mode line, or have the parent record the extension.

3. **NIT, scope.** Location: `internal/boardengine/beammodes.go` `modeHeader` (diff 002, lines ~134–147).
   - **Summary:** G4 asks for the hub-wide value at the top "when it applies to every beam". The header states it unconditionally, even when no beam takes it.
   - **Evidence:** `modeHeader` returns the "Hub-wide default:" line whenever `b.modes != nil`.

No other change lacks a trace, and no other decision lacks a change that builds it.

Tool calls:
1. `grep` of `projectSection` in `internal/boardengine/render.go`, to check whether README sections reach nested projects, which bears on whether the project-file mode line (finding 2) is needed. The reading manifest did not cover `projectSection`'s body.
