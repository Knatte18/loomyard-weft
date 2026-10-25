```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. Per behaviour, the test that pins it, or none.

**Behaviour → pinning test**
- **Window title from the display.** Pinned in the real-tmux window-order test (`ReplacingTheOverviewWithATitleKeepsItFirstUnderTheTitleThenAgentsInStartOrderThenShell`, `windoworder_integration_test.go:298`). The rows asserting `overviewWindowName` for an untitled strand (lines 226 and 268) cover the empty-Title fallback. Nothing unit-tests `overviewWindowTitle` directly, and that is fine.
- **`runsWindowTitleOf`.** Pinned by `TestRunsWindowTitleOf` (`start_test.go:352`). The runs strand spec carries `Title: "beam: prime"` in `TestStart_EnsuresTheRunsStrandBeforeTheOrch` (`start_test.go:251`).
- **`runsWindowTitle` set from `location.WorktreeName` and `beam` at `orchcli/cli.go:194`.** None. Both tests hard-code the field or call the helper directly. This is a small wiring line, so I rate it a NIT.
- **Run cells (paused minutes, Run-Shed child stage, `status unreadable`, `starting`).** Pinned by `TestOverviewRunCells` (`overview_test.go:56`). `TestDobbyRunCells_ReadsTheRunsFromDisk` (`overview_test.go:324`) covers reading them from disk.
- **`DobbyLines` layout.** Covered by `TestDobbyLines`: the dobby lines, run cells in the pane's column widths, the unreadable run's `status unreadable` line, and the empty-cells fallback.
- **Frame order: dobby, blank line, orch.** `TestOverviewSource_DrawsTheDobbyAboveTheOrchs` covers a frame with the dobby alone, then with an orch line below it.
- **Dobby-less worktree draws the orch section alone.** `TestOverviewSource_DrawsTheOrchLine` runs with `dobby` nil and asserts orch lines only.
- **No run drawn twice.** There is no explicit test. The run list is removed, so a regression would have to reintroduce a list. I do not count this as a gap.
- **Orch scope.** `TestOrchsAtOrBelow` has rows for the prime, a beam with beams below it, a nested beam, a leaf beam, and a `broken` parent chain that is left out. `TestOverviewSource_DrawsTheOrchLine` covers the labels, the list-failure fallback and the beam's fallback label.
- **Cyclic parent chain.** None, and `atOrBelow` (`overview.go:395`) has no cycle guard. This is the code lenses' concern. For this lens it is an uncovered case of a focus question. See finding 1.
- **`lyx beam status` prints the pane's lines, run cells included.** None. `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` (`status_integration_test.go`) was not changed by the diff. It checks only the header line of dead dobbies, with no runs. `dobbyStatus` now builds `NewDobbyRunCells` and passes `cells.Of` into `DobbyLines`, and nothing exercises that wiring. See finding 2.
- **The cancel integration test.** Its check that a run is gone now reads the prime's dobby view. `cancel_integration_test.go` changed (26 lines) and I did not open it; the changed lines are outside what my context holds.

**Findings**
1. MEDIUM, design. `overview_test.go:536` (`TestOrchsAtOrBelow`) and `overview.go:395` (`atOrBelow`). No row or test covers a cyclic parent chain. The focus directive asks whether a cyclic or missing parent chain terminates. The missing chain is covered by the `broken` location, whose `parentOf` returns an error. The cyclic case is not covered. If it is a real hang, the handler should raise it as a correctness defect, since a cyclic chain makes `atOrBelow` loop forever.
2. LOW, design. `internal/beamcli/cli.go:87-110` (`dobbyStatus`) against `status_integration_test.go:21`. The behaviour the diff adds, that `lyx beam status` prints run lines with cells, has no test. The existing test has no runs, so it passes with the cells wiring broken or removed. A row asserting a run line with its state and stage would pin the acceptance criterion "`lyx beam status` prints the same dobby lines the pane draws".
3. NIT, design. `internal/orchcli/cli.go:194`. The wiring from worktree name and beam flag to `runsWindowTitle` is untested.

**Test economy**
- `TestRunsWindowTitleOf` duplicates nothing: `TestStart_EnsuresTheRunsStrandBeforeTheOrch` hard-codes the prime title only.
- No new test duplicates an existing one. The new `AnUnreadableRunTakesNoPartInTheColumnWidths` row and the orch assertions added to `TestOverviewSource_DrawsTheDobbyAboveTheOrchs` pin behaviours no other test covers.

**Tool calls**
1. A grep of `overview_test.go` test names and of the repo's tests for orch-scope, title and `DobbyRunCells` symbols. My context held only the diffs of three test files.
2. `sed -n 440,576p overview_test.go`, plus a grep for beamcli status tests. I needed the bodies of `TestOrchsAtOrBelow` and the orch-line test, and the beamcli test names.
3. `sed -n 15,120p status_integration_test.go`, plus a grep for reed title tests. I needed to see what `lyx beam status` is asserted against.
