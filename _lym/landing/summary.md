# Batten commits its driver records; a yielded step saves no envelope

Fixes issue #43: a hub's prime collected thousands of untracked files under `_lym/shed/<slug>/`, all written by the Go driver of a `lym batten run`.

## Cause

Two faults combined:

- **One envelope per pass.**
  `sheddriver.Run` saved every step's stop envelope under `drive-reports/` before checking whether the step yielded.
  While batten's Run-Shed row watches a child run, dobby starts a pass about every 30 seconds and each pass yields, so each pass left one `drive-envelope-*.json` that nothing reads.
- **Batten never committed its driver directories.**
  `battenRunCommitPaths` staged the status, seed, notice marker, settle records and child-recipe record, but not `drive-reports/`, `tackler/` or `arbiter/`.
  Loom's `statusCommitPathspec` stages all three; batten's did not, so every stop report, envelope and agent attempt in the prime stayed untracked.
  Batten also wired no `CommitRecords` seam and its park committed nothing, so a stop report written after the transition's commit was never staged.

## Changes

- `internal/sheddriver`: the yield check now runs before `SaveEnvelope`, so a yielded step saves no envelope.
  A yield already decided nothing and wrote no settle record or report; now it writes no file either.
- `internal/shedrun`: `DriverRecordsRel` returns the anchor-relative drive-reports, arbiter and tackler directories that hold a file.
  An empty directory is left out, since a pathspec matching nothing fails the commit.
  Loom's `statusCommitPathspec` uses it in place of its three inline checks, so both recipes stage the same driver records.
- `internal/battencli`:
  - `battenRunCommitPaths` appends `DriverRecordsRel`, so every status transition carries the driver records with it.
  - The new `commitRunRecords` commits the run's records outside a transition and starts their push.
    It skips the commit while the pair is mid-merge and logs a push that fails to start.
  - Batten's driver now wires `CommitRecords` at a done stop, and its hand-back `Park` commits the records before writing the park marker.
    A failed commit at the park is logged and the run is still parked, as loom's park does.

## Design decisions

- Drive reports stay durable under `_lym`, where `shedrun` declares them and where loom commits them.
  The issue offered moving them to `.lym/` if they were transient.
  They are durable: `reportengine` counts drive reports from the committed run records, and the tackler and arbiter attempts are what a resumed machine reads.
- Batten's `status.json` still shows as modified while Run-Shed watches a child.
  `battencli/commitstatus.go` documents this as a deliberate trade: the no-op-transition skip declines one commit per self-bounce.
  The file is tracked, and the next real transition commits it.
- Files already left untracked in a hub are not migrated.
  The next commit of that run's records stages its driver directories whole, including the old envelopes.
  An operator who wants the old envelopes gone can delete them by hand first.

## Tests

- `TestGoDriver_Settle`'s yield row asserts that no envelope is saved.
- `TestBattenRunCommitPaths` covers the driver directories: a directory holding only an empty attempt directory is left out, and one holding a file is staged.
- The new `TestCommitRunRecords` covers commit-and-push, nothing committed, mid-merge, a failed merge-state read, a failed commit and a push that fails to start.
- `TestBattenDriverDeps` now requires the `CommitRecords` seam.
- `battenCLI` gains a `recordsDeps` override, which the park and `CommitRecords` seams read through `runRecordsDeps`.
  The run tests' fake receiver sets stubs that find nothing to commit, since its temp directory is no git pair.
