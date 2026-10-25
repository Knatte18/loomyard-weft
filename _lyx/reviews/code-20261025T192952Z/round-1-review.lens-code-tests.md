```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, covering which test pins each behaviour the parent change `afd030a47` adds.

**Per behaviour: pinning test**
- **Runs window named `beam: prime` / `beam: <slug>`:** `TestRunsWindowTitleOf` (`internal/orchcli/start_test.go`) pins the name function. The strand spec carrying the title is pinned by the runs-strand spec assertion in the same file, `render.Display{Anchor: render.AnchorOverview, Title: "beam: prime"}`.
- **Window title reaches the tmux window order:** the reedengine integration subtest `ReplacingTheOverviewWithATitleKeepsItFirstUnderTheTitleThenAgentsInStartOrderThenShell` (`internal/reedengine/windoworder_integration_test.go`).
- **`DobbyLines` takes per-run cells (state, stage, in-stage, paused):** `TestDobbyLines` rows `AnAliveDobbyShowsItsRunsWithTheirCellsAndWhyEachTaskWaits` and `ADeadDobbyNamesTheWayForwardAndARunWithNoPassSaysSo` (`internal/battencli/overviewdobby_test.go`).
- **An unreadable run is left out of the column widths:** row `AnUnreadableRunTakesNoPartInTheColumnWidths` (same file).
- **Window frame is the dobby section above the orch lines, with and without orch state:** `TestOverviewSource_DrawsTheDobbyAboveTheOrchs` (same file).
- **Orchs shown per window, and the fallback to the beam's own orch:** `TestOrchsAtOrBelow` and the fallback-frame check in `internal/battencli/overview_test.go`.
- **`lyx beam status` carries each run's cells:** `status_integration_test.go` in `internal/beamcli` seeds a run and asserts the collapsed run line.
- **`DobbyRunCells` reading the pause record and the child's status and seed:** pinned by `internal/battencli/overview_test.go`, which the merge rewrote; I did not open its individual rows.
- **`HubOrchs` resolving each pair through `lyxcwd.ResolveWorktree`, and skipping a pair that does not resolve or whose origin does not read:** none that I saw in the diff. The merge's test diffs touch only `orchcli/start_test.go`, not a `live_test.go`. A pre-existing test may cover it, but my context does not show one.
- **`c.runsWindowTitle = runsWindowTitleOf(location.WorktreeName, beam)` wiring (`internal/orchcli/cli.go`):** none. The strand-spec test sets `c.runsWindowTitle` by hand, so a wrong `beam` flag or worktree passed in `cli.go` would not fail any test.

**Findings**
1. **LOW, design (uncovered behaviour):** nothing pins the `runsWindowTitle` wiring in `internal/orchcli/cli.go`, the `runsWindowTitleOf(location.WorktreeName, beam)` call. `TestRunsWindowTitleOf` only calls the pure helper, and the spec test injects the title directly. Evidence: `internal/orchcli/start_test.go` sets `c.runsWindowTitle = "beam: prime"` directly.
2. **NIT:** `HubOrchs`'s skip-and-warn paths have no test I could find in the merge's diff. Confirm a pre-existing test covers them before treating this as a gap.

**Test economy:** no duplicate tests, no per-helper test that a public-surface test already covers, and no new assertion on a stencil's static prose. The helper test `TestRunsWindowTitleOf` is the only pin on the title function, so it is not redundant.

**Tool calls (1 of 6):** one `git diff 4e7f23e48 b244f3e14` over the four merged test files (`orchcli/start_test.go`, `beamcli/status_integration_test.go`, `reedengine/windoworder_integration_test.go`, `battencli/overviewdobby_test.go`), because my context held only the production-code diff.
