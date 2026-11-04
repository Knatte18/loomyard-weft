# batten: commit driver records; no drive envelope per yielded poll

`lym batten`'s driver wrote its records under `_lym/shed/<slug>/` (`drive-reports/`, `tackler/`, `arbiter/`) but never committed them, so a hub's weft checkout stayed permanently dirty and the records were lost with the checkout.
The driver also saved a stop envelope on every pass, including passes that yielded, which is why a run waiting at a row produced one `drive-envelope-<timestamp>.json` roughly every 30 seconds.

This change fixes both causes.

Closes #43

## Changes

- `sheddriver`: a step that yields now ends the pass before its envelope is saved, so a wait at a row writes no file.
  Only the envelope of a stop the driver decides is saved.
- `battencli`: the run's records commit now stages the driver's `drive-reports`, `tackler` and `arbiter` directories, each when it holds a file.
  The driver writes a hand-back's stop report and a done step's envelope after the status transition's own commit, so two stops commit them explicitly through a new `commitRunRecords`:
  the hand-back's `Park` seam, and the new `CommitRecords` seam at a done stop.
  A merge in progress on the pair commits nothing, and the next commit carries the records.
  A failed commit at a hand-back is logged and the run still parks.
- `sheddriver`: a done stop with `friction: failed` calls `CommitRecords` a second time after its hand-back report is written, so that report is committed too.
- `shedrun`: new `DriverRecordsRel` returns the driver-record directories that hold a file, and `HoldsFile` is now exported from there.
  `loomcli`'s statusCommitPathspec and the landing code use these, replacing `loomcli`'s private copy of `holdsFile`.

## Design notes

- The directories stay under `_lym/` (tracked) rather than moving to `.lym/`: the tackler and arbiter records and the stop reports are durable run history that should sync with the run, and the per-poll volume was the actual problem.
- Each directory is staged only when it holds a file, because a pathspec that matches nothing fails the commit.
- `CommitRecords` must be a no-op over records already committed, since the friction path calls it twice.

## To check by hand

- Run `lym batten` through a wait at a row and through a hand-back, then confirm `git status` in the weft checkout is clean and no `drive-envelope-*.json` accrues per poll.
- `battencli`: the `Park` seam's commit and `CommitRecords` are covered by seam-level tests with a fake `recordsDeps`; the real fabric commit over a git pair is not exercised by them.
