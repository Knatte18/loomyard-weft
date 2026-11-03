```yaml
lens: code-tests
outcome: reported
```

**Task:** the code-tests lens. For each behaviour the diff adds or changes, the test that pins it.

**Per behaviour**
1. `drivable` and the uncovered branch of `planPass` (paused with `ReachedParent` gives a full pass): pinned by `TestPlanPass/PausedReachedParentGivesFull` (`passes_test.go:39`). `PausedGivesNone` (line 37) pins the unreached paused case, and `HaltedReachedParentGivesNone` (line 38) pins that a halted run is not drivable.
2. The `run.State != runStateRunning` clause in the covered branch (`passes.go:133`): pinned by `PausedReachedParentCoveredGivesNone` (line 45). Without the clause, a covered paused run at Run-Shed would return `passContinue`.
3. The `run.State != runStateRunning` clause in the uncovered branch (`passes.go:138`): pinned by `PausedReachedParentGivesFull`. A run at Run-Shed with no owed pass and no origin-owed marker would otherwise get a continue pass.
4. Lift gives a full pass for a covered paused reached run: pinned by `PausedReachedParentCoveredLiftGivesFull` (line 46).
5. `OriginOwed` on a paused reached run gives `passFullNoReopen`: pinned by line 40. It would still return a full pass without the new clause, because `OriginOwed` alone forces it. The row therefore pins only the no-reopen variant.
6. Cancel marker on a paused reached run gives none: pinned by line 47. It checks the existing guard, and `CancelMarkedGivesNone` already covers that guard for a running run.
7. `drivePasses` `liftPending` cleanup changed from `State != running` to `!drivable`: **none**. The only test that exercises it, `passes_test.go:521`, uses `"blocked"`, which gives the same result under the old and new condition. No `drivePasses`-level test uses a paused reached run, so reverting the line fails nothing.
8. `battencli.DobbyRuns` filling `ReachedParent` (the paused-only gate, the `hubChildReachedParent(prime, slug)(status, true)` call, and the Warn-and-leave-unreached path): **none**.
   - `dobbypairanchor_test.go` calls `DobbyRuns` (lines 92, 119, 149, 215) but I found no paused status or `ReachedParent` assertion in it.
   - The only other test file that mentions `DobbyRuns` or `DobbyViewReader` is `cancel_integration_test.go`, and it has no paused-state dobby case either.
   - The summary says the judgment is pinned by `TestChildReachedParent`. That test covers `childReachedParent` (`cancel_test.go:950`), not the wiring.
   - So the wiring is untested. Removing the read, passing a wrong `found` argument or dropping the paused gate fails no test.
9. `planPass` with `Held` or `OwnedElsewhere` on a paused reached run: no row. The code reaches these branches after the shared `drivable` check, and the summary claims they behave as for a running run (`summary.md:12`). `OwnedElsewhere` is the one with new reach, since a paused reached run owned elsewhere now gets a full pass on probe ticks.
10. `RunView.ReachedParent` read by `DobbyViewReader.Read` and `BuildView`: no view-level test. The field changes nothing in `BuildView`, so there is no behaviour to pin.
11. The risk the summary names (a pass that exits 0 but leaves the run paused re-launches each probe tick): no test pins or bounds it.

**Findings**
- **MEDIUM, design:** the `DobbyRuns` `ReachedParent` read has no test, at `internal/battencli/dobbyreads.go:185-196`.
  - It is the only code path that sets `ReachedParent` in production, and the behaviour depends on it.
  - A test in `internal/beamcli/dobbypairanchor_test.go` or `internal/battencli` should build a paused batten status whose child reached the parent and assert `ReachedParent` is true. A non-paused run and an unreadable judgment (Warn, unreached) should leave it false.
- **LOW, design:** the `drivePasses` `liftPending` change at `passes.go:186` is not pinned.
  - `passes_test.go:521` uses `"blocked"`, which passes whether the condition is `!= running` or `!drivable`.
  - A paused reached run that is covered, lifted, and then held or in flight would pin it.
- **LOW, design:** there is no `TestPlanPass` row for a paused reached run with `OwnedElsewhere` set, or with `Held` set. Both branches now follow the same rules, and `OwnedElsewhere` has new reach.

**Tool calls (1)**
- `grep` of the test files for `DobbyRuns`, `DobbyViewReader` and paused statuses. My context did not show whether any test fed `DobbyRuns` a paused state.
