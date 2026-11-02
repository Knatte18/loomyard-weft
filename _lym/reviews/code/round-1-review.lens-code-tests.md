```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour**

1. `Cancel` on a terminal request with a live owned entry lifts the entry, cancels nothing and leaves the file as it was. Pinned by the new `verbs_test.go` row over `PhaseDone`, `PhaseFailed` and `PhaseCancelled`.
2. A terminal request never gets its daemon killed. Pinned by the `len(w.killed) != 0` check in the same row and in the old terminal row.
3. A terminal request with no `Owned` reports `Lifted == nil`. Pinned by the old terminal row, which now asserts `report.Lifted != nil`.
4. A stop on a shared branch lifts the hub entry, holds each stopped run and names the stop before the runs. Pinned by the rewritten `watch_test.go` row, which checks that only the `p1` entry stays live and asserts the notice text.
5. A stop with no runs returns the stop sentence alone, the `held == ""` branch of `reconcileAtWake`. Pinned incidentally by the over-bound notice row, which sets `Stop` with no `Runs`.
6. A notice over `noticeLineBound` goes to the notice file and a short line naming the file is queued. Pinned by the over-bound row, which checks length, the file contents and the file name in the notice.
7. A refused notice is retried after a doubling wait. Pinned by the row with two refusals, which expects waits of one and two seconds.
8. A notice refused through the whole retry window returns the error and the entry stays lifted. Pinned by the window row. Its `waited` bounds also constrain the 30-second ceiling.
9. `advance(PhaseResuming)` failing at the wake lifts the entry through `kept`, sends one `fail` notice and returns the error. Pinned by the row that replaces the request file with a directory after the hibernate.
10. `liftOwned` failing at the wake goes to `fail`. Not covered by a new row. I did not check whether an older row covers it, so I'm not raising it.
11. `reconcileAtWake` returning an error now passes `kept` to `fail` where it passed `nil`. No row separates the two, since `Owned` is normally set. It is the same defect as finding 3 below.

**Findings**

1. **MEDIUM, design.** Location: `internal/hibernateengine/watch.go:166-169` (`sendNotice`, long-line branch). The multi-line condition (`strings.Contains(line, "\n")`) has no row. The over-bound row uses a long single-line text only. A `Stop` or reconcile error containing a newline, under 1 KB, would go uncovered. Fix: add a short multi-line `Stop` row asserting the notice file holds the whole text and the queued line is a single line.
2. **MEDIUM, design.** Location: `internal/hibernateengine/watch.go:185-194` (`writeNoticeFile` failure branch). No test makes the file write fail, so the sentence "could not be written ... in the daemon's log" and the fallback that still queues a short line are unpinned. Fix: add a row where `NoticePath` sits under a regular file, asserting the queued line is short and names the failure.
3. **LOW, design.** Location: `internal/hibernateengine/watch.go:146-153` (`target.Owned = kept` when `r.Owned` is nil) and `:142-145`. No row has a request with no `Owned` and a `kept` entry reaching the wake. The `nil` to `kept` change on the reconcile-error path is likewise unpinned. Fix: add a row, or drop the branch if it is unreachable.
4. **LOW, design.** Location: `internal/hibernateengine/watch.go:154-160`. A `finish(PhaseDone)` write failure after a successful lift is untested. The only write-failure row breaks the file before `advance`, so it never reaches this line. The behaviour is that the wake notice is still sent with the lifted outcome and the write error is joined into the return. Fix: add a row that breaks the file after the `PhaseResuming` advance, for example from `world_test.go`'s `Reconcile` hook.
5. **LOW, design.** Location: `internal/hibernateengine/watch.go:298` (`fail` with the short headline). No row has a long failure reason, so the headline and the notice file in the `fail` path are unpinned. Fix: add a row with a long `Reconcile` error message or a long `TenderStatus` error, asserting the short line and the file.
6. **LOW, design.** Location: `internal/hibernatecli/deps.go:117` (`NoticePath` wiring to `hibernate/notice.txt`). The engine tests use a fake path, so nothing pins the real wiring. I did not check `cli_integration_test.go`, so this may be covered there; if `cli_integration_test.go` does not assert it, add one assertion on the path there rather than a new test.

**Tool calls:** none. My context already held the diff and every test hunk I judged.
