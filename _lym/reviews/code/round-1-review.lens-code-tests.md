```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. Per behaviour, the test that pins it, or none.**

1. **Yield saves no stop envelope** (`sheddriver/drive.go:204-210`).
   - Pinned by `TestGoDriver_Settle`, the "a yielded step ends the pass…" row with `wantNoEnvelope` (`drive_test.go:228-229`, assertion at ~858). The row would fail if the early return moved back after `SaveEnvelope`.
   - Its `wantCalls: ["step"]` also shows that nothing else ran.

2. **Yield with `OK`, `LoopState` running and `Outcome` yield still ends with `ExitYield`.**
   - Pinned by the same row's `wantExit: ExitYield`.

3. **A non-yield stop still saves its envelope after the move.**
   - Every other `Settle` row sets `stop.EnvelopePath`, and the stop reports quote it.
   - No row asserts the positive save count. The `envelopes` counter is only checked when `wantNoEnvelope` is true.
   - No finding. The existing report rows would fail if the save were dropped.

4. **`DriverRecordsRel`: order, file-only inclusion, and an empty directory omitted** (`shedrun/paths.go`).
   - Pinned indirectly by `TestBattenRunCommitPaths` in `battencli/commitstatus_test.go`.
   - It covers an empty tackler attempt directory being omitted.
   - With files present, it asserts the three rels in the order drive-reports, arbiter, tackler.
   - The loomcli `statusCommitPathspec` rows in `wiring_commitstatus_test.go` (about lines 251-343) still pin the loom side.
   - There is no direct `shedrun` test, and none is needed because the public callers' tests cover it.

5. **`dirHoldsFile` on a missing or unreadable directory returns false.**
   - A missing directory is pinned by the "status alone" rows of `TestBattenRunCommitPaths`.
   - The unreadable-directory branch has no test.
   - It is not a finding: the behaviour is unchanged from the old `holdsFile` walk, and an unreadable-directory test is hard to run portably.

6. **`commitRunRecords`** (`battencli/commitstatus.go`).
   - Pinned by `TestCommitRunRecords`, which has a row for each branch:
     - commit plus push start
     - nothing committed, so no spawn
     - merge active, so no commit
     - merge-state read error
     - commit error
     - spawn error not returned
   - It also checks the commit message.

7. **`CommitRecords` wired in batten** (`battencli/drive.go`).
   - Pinned by `TestBattenDriverDeps`, which moved `CommitRecords` into the wired set (`driverdeps_test.go:63`).
   - That test only checks the seam is non-nil.
   - The `sheddriver` "done commits…" rows pin that the driver calls `CommitRecords` at a done stop (`drive_test.go:226` and others).
   - Nothing pins that batten's closure actually calls `commitRunRecords` with its slug.
   - **Finding below.**

8. **`Park` commits the records first and still parks when the commit fails** (`battencli/drive.go:276-279`).
   - **No test.** `driverdeps_test.go:68` only asserts `HandBack.Park != nil`.
   - The `run_test.go` stub makes the commit a no-op, so no test reaches the `Park` commit path.
   - Neither "Park commits" nor "Park parks despite a commit error" is pinned.

9. **`runRecordsDeps` uses the `recordsDeps` override when set** (`battencli/drive.go`).
   - It is exercised only as test plumbing through `run_test.go`.
   - No finding.

10. **`battenRunCommitPaths` appends the driver records.**
    - Pinned by `TestBattenRunCommitPaths`, as in item 4.

11. **`loomcli` `statusCommitPathspec` refactor to `DriverRecordsRel`: same directories, same order.**
    - The existing `wiring_commitstatus_test.go` rows pin it (line 319 asserts the drive-reports rel in its position).
    - Nothing new is needed.

**Test economy.**
- `TestCommitRunRecords` and the `TestBattenRunCommitPaths` additions duplicate no existing test.
- The `TestBattenRunCommitPaths` additions extend the existing test, as they should.
- No redundant new test.

**Findings**

- **MEDIUM, class `design`, `internal/battencli/drive.go:276-279` (evidence `driverdeps_test.go:68`, `run_test.go:48-55`).** The new Park-seam behaviour has no test. It commits the run's records before parking and still parks on a commit failure, which is the focus directive's "Park still parks on every commit failure". No row would fail if the commit were removed, or if the error became a returned error that skips the park marker. Add a test that calls `deps.HandBack.Park` with a `recordsDeps` stub, once with a successful commit and once with a commit error. It should assert that the commit was attempted and that the park marker exists in both cases.
- **LOW, class `design`, `internal/battencli/drive.go:327-329` (evidence `driverdeps_test.go:63`).** The `CommitRecords` closure is pinned only as non-nil. Nothing asserts that it calls `commitRunRecords` for the run's slug. The same `Park` test can add this by invoking `deps.CommitRecords()` with a `recordsDeps` stub that records the commit message and checks it names the slug.

**Tool calls (1):** a grep for tests referencing `DriverRecordsRel`/`dirHoldsFile`, `Park`, and `commit-records`. The manifest lacked the test-side call sites.
