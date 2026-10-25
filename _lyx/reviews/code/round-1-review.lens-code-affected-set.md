```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each changed symbol, output or behaviour, I list what reads or asserts it and whether the diff covers it.

**Set per change**
- `DobbyLines(label, view, runCells, now)`, new signature. Callers are `internal/battencli/overviewdobby.go:80`, `internal/beamcli/cli.go:110` and `TestDobbyLines`. `beamcli` builds `NewDobbyRunCells` once per status call (`cli.go:91`). Covered; no stale caller.
- Removed symbols: `renderOverview`, `overviewLineOf`, `overviewShows`, `runsLines`, `hubOrchPaths`, `overviewOrchLine`, the old `newOverviewSource`, `overviewNoRuns` and `overviewRunsBroken`. A grep for these in non-test code finds no reference left. The tests and the cancel integration test were rewritten (the diff touches `cancel_integration_test.go`). Covered.
- `newOverviewSource(prime, worktree, thresholds, now)`. Callers are `overview.go:512` and the tests (`overview_test.go:449,532`, `overviewdobby_test.go:103`). Covered.
- `runsStrandAddSpec(cmd, title)`. The single caller (`runs.go:66`) passes `c.runsWindowTitle`. `start_test.go:165` sets the field in the harness and `start_test.go:249` asserts the display. Covered.
- `runsWindowTitle` is set in `orchcli/cli.go:194`, after `location` and `beam` are known. Covered. I did not check whether any other path builds an `orchCLI` and reaches `ensureRunsStrand` without it. If one does, the title is empty and the window falls back to `overview`, which is a safe default.
- `render.Display.Title` (new JSON key `title`, `omitempty`) is read only in `windoworder.go` (`overviewWindowTitle`) and used at `spawn.go:282`. Reed finds windows by id, the `@lyx_own` and overview markers, and `strands.go` names; no `-t <name>` lookup targets the window name. The only name comparisons are in the integration test (`windoworder_integration_test.go:205,226,240,268`), which still compares against `overviewWindowName` for untitled overview strands. Covered.
- `lyx beam status` output: the `Long` help, `beamcli/doc.go:82` and `docs/overview.md:444` were updated to the new content. Covered.
- Docs the scope lists:
  - `orchengine/doc.go:13`, `reedengine/doc.go` (the overview-anchor paragraph), `beamcli/doc.go`, `battencli/cli.go:20-30` and `docs/overview.md:633-637` are updated.
  - The orch role stencil `contracts/stencils/orch/orch-template-role.md` has no mention of the overview window or the runs strand (grep finds none), so there was nothing to edit.
  - `orchcli/start.go:358` help text updated.

**Findings**
1. LOW / consistency — `docs/overview.md:410`, the orch module row, was not updated. It still reads "`start` first ensures the worktree's `runs` strand, which runs `lyx batten overview` as the session's overview window … the session's windows run overview, agents, shell." With the Title change the window is named `beam: <slug>`, not `overview`. The scope names `docs/overview.md` as a doc to update and the diff edits only lines 444 and 633. Fix: say the first window is the beam's window, named `beam: prime` or `beam: <slug>`, in place of "overview". This is a small docs gap; the sentence is still true of the anchor ("overview" is also the anchor name).
2. No other member of the affected set is left wrong. The strand sidecar, the relaunch path and the persisted `title` key all tolerate an older display without `Title` (empty falls back to `overview`).

**Tool calls**
1. Grep and diff for callers of `DobbyLines`, `NewDobbyRunCells` and `newOverviewSource`, doc mentions of the window, and removed symbols. My context did not hold the caller and test sets.
2. Grep of stencils, `docs/overview.md`, `reedengine/doc.go` and `orchengine/doc.go` for "overview", name-based window lookups, and the diffs of `docs/overview.md` and `reedengine/doc.go`. I needed to confirm what the doc landings changed.
3. Grep for other mentions of the runs strand or the first window, plus `battencli/cli.go:18-30`. I needed to find stale doc lines.
