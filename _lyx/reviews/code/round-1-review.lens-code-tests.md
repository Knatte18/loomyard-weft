```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it (or none), and I apply test economy.

**Per behaviour: pinning test**

1. **`BuildView` reasons (dependencies, owned elsewhere, paused, ready).** Pinned by `view_test.go` `EachEntryReasonIsListed` and `AReleasedTaskWithNothingHoldingItIsReady`.
2. **Refusal text and next try shown, with the fingerprint check and backoff expiry.** Pinned by `view_test.go` `ARefusedStartShowsItsTextAndTheNextTryUntilTheBackoffRunsOut`. It has rows for backing off, due and entry-changed.
3. **`max_runs` in board order, slots freed by owned-elsewhere or done runs.** Pinned by `MaxRunsHoldsBackTheTasksPastTheFreeSlotsInBoardOrder`, `MaxRunsZeroHoldsBackEveryTask` and `RunsShowTheirRecordAndDoneOrOwnedElsewhereRunsTakeNoSlot`.
4. **Runs section (done filtered out, pass in flight, last pass exit, failing count, sorted by slug).** Pinned by `RunsShowTheirRecordAndDone…`.
5. **Only tasks released to this worktree wait (other parent, draft, project).** Pinned by `OnlyTasksReleasedToThisWorktreeWait`.
6. **`Alive` false.** Pinned by `ADeadDobbyReadsNotAlive`.
7. **`released` / `heldByEntry` / `readyToStart` refactor.** Pinned by the existing `TestReadyToStart`. Its fixture was updated to `PendingDependencies`, and the `OpenDependencyDoesNotStart` row now uses it.
8. **`slotsTaken` shared by `runsInFlight`.** Pinned by the existing `max_runs` subtests in `starts_test.go`, including "a run marked owned elsewhere leaves a slot under max_runs free". No new test is needed.
9. **Refusal text written by the refused start.** Pinned by the new assertion in `starts_test.go:109`, `Starts["a"].Refusal == "pre-run refused"`.
10. **Refusal text cleared when a pass claims the task or its entry changes.** **None.**
    - The diff changes the clearing condition in `updateState` (`starts.go`, `st.Refusal == ""` in the delete case).
    - The existing entry-changed step (`starts_test.go`, the `a-v2` retry) never asserts that `Refusal` is empty or that the record is gone afterwards.
    - No row asserts the record is deleted after a claim.
    - If the `st.Refusal == ""` term were dropped, a record holding only a refusal would be deleted early, and no test would fail.
    - The record is cleared by `*s = StartState{}` in `refusedStart`, and elsewhere by the pre-existing clear. I judged the new delete condition from the diff only, with no further tool read.
11. **`Log` seam: one line per start and per pass, with time, kind and slug.** Pinned by `passes_test.go` `TestLog_OneLinePerStartAndPass`. It covers `pass full`, `pass continue` and `start`.
12. **`passKind.String()` names `full-no-reopen`, and `Log == nil` logs nothing.** **None** for `full-no-reopen`, and **none** for the nil seam. Every other harness test runs with a nil `Log`, so the nil case is implicitly exercised without a panic. `full-no-reopen` is the only unasserted kind.
13. **Hub wiring `Log` pointed at stdout (`beamcli/dobbydeps.go`, `fmt.Fprintln(logOut, …)`, `out` passed from `cli.go:204`).** **None.** No test in `dobby_test.go` or `dobby_integration_test.go` asserts a log line reaches the writer. This is a thin one-line wiring.
14. **`DobbyLines` rendering (header alive or not, runs, each wait reason, ready, no runs, pass cell, first line of the error, minutes).** Pinned by `TestDobbyLines`, three subtests. `AnAliveDobby…` covers pass in flight, last pass minutes, full pass owed, failing count, ready, dependencies plus pause, refusal plus retry minutes, owned elsewhere and `max_runs`.
    - Uncovered: the `owned by another machine` note in the run pass cell, and `dobbyUnreadableFormat` (`overviewDobbyLines` on a read error).
15. **Overview frame draws the dobby below the runs after a blank line, at the frame's time, and nil draws nothing.** Pinned by `TestOverviewSource_DrawsTheDobbyBelowTheRuns` for the draw case. The nil case is covered by the existing overview frame tests, which leave `dobby` nil.
16. **Dobby section appears only in prime and beam panes, never a task pair's.** **None.**
    - The wiring in `overviewCmd` uses `NewDobbyViewReader(location, acting)`, and nothing asserts the pair case.
    - A pane run from a task pair is neither pinned nor refused by any test.
    - `NewDobbyViewReader` is also untested directly.
    - This is an acceptance criterion: "a task pair's frame carries none".
17. **`DobbyTasks` change (`PendingDependencies` by slug, parent and owner read for every empty-status task).**
    - **No direct test.** The moved `DobbyTasks`, `DobbyRuns`, `DobbyLandings`, `DobbyParents` and `DobbyViewReader.Read` have no test in `battencli`.
    - `battencli` tests only mention `dobbyRuns` as a local variable name in `continue_pass_test.go`.
    - The only hub-level coverage is `status_integration_test.go`, which runs with an empty board and so exercises no pending dependency and no parent read.
    - Behaviour with no test: a dependency on a pending slug being listed, a pruned dependency being skipped, and a parent being read for a task whose dependencies are open.
    - The removed beamcli reads had no listed tests in the diff either, so this is a gap in the changed behaviour rather than a regression.
18. **`lyx beam status` covers the prime and every local beam, with the view in its envelope.** Pinned by `status_integration_test.go`. It covers the prime plus one beam, in order, with `alive=false` and the header line.
    - Unpinned: `alive=true`, per-entry error and joined failure, the non-ok path, and running from the prime. It runs only from the beam.

**Test economy findings**

- **Finding 1. MEDIUM, design.** Location: `internal/dobbyengine/starts.go` (`updateState` delete case) and `internal/dobbyengine/starts_test.go:109-134`. The refusal text's clearing (a claim, an entry change) has no assertion, though it is an acceptance criterion. Add a row or assertion in `TestStartReady_RefusalsAndBackoff` that `Starts["a"].Refusal` is empty, or the record is gone, after the pass claims the task and after the entry changes.
- **Finding 2. MEDIUM, design.** Location: `internal/battencli/dobbyreads.go:209-258`. The `DobbyTasks` behaviour change has no test: `PendingDependencies` by slug, parent and owner read for every empty-status task, a pruned dependency skipped. Add a tier-appropriate test over a board fixture, or a `status_integration_test.go` row with a task that has an open dependency.
- **Finding 3. LOW, design.** Location: `internal/battencli/overview.go:536-541`. "The dobby section appears only in prime and beam panes, not in a task pair's frame" has no test. The `overviewCmd` wiring and `NewDobbyViewReader` are unpinned for the pair case.
- **Finding 4. LOW, design.** Location: `internal/dobbyengine/passes.go:108-119`. `passKind.String()` for `full-no-reopen` is unasserted, and `TestLog_OneLinePerStartAndPass` covers only `full` and `continue`. Add a `full-no-reopen` pass to that test.
- **Finding 5. LOW, design.** Location: `internal/beamcli/dobbydeps.go:148`. The hub `Log` wiring to the writer is unpinned.
- **Finding 6. NIT, consistency.** Location: `internal/battencli/overviewdobby.go`. The unreadable-line branch (`overviewDobbyLines` error) and the `owned by another machine` note in `dobbyRunPass` are unasserted. Add a `TestDobbyLines` row for the owned-elsewhere run.

**Redundancy:** I found no new test that duplicates another. `view_test.go` is the only home of the `max_runs` view logic, and `starts_test.go` covers the scheduler's side. `status_integration_test.go` pins the beam enumeration and envelope, not rendering, so it does not duplicate `TestDobbyLines`. Nothing is redundant.

**Tool calls:**
1. `grep` for the moved read symbols and the `Refusal` / `Starts[` uses in tests. My context did not list which tests mention the moved reads.
2. `grep` in `continue_pass_test.go`, a `sed` of `starts_test.go:100-175`, `git diff --stat` of the test files, and `ls internal/beamcli/*_test.go`. I needed the surrounding refusal and `max_runs` tests, which the diff hunks cut off, and the list of changed test files.
