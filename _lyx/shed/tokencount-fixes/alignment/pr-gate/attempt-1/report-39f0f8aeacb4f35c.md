---
verdict: APPROVED
---

## Alignment at PR-Gate: tokencount-fixes, PR 630 at d017696f7

The diff against `main` changes `tools/tokencount` alone, plus the package's row in `cmd/lyx/testdata/test-budget.yaml` (16 to 17).
The decision record holds no `Added after Discussion` entry, so the board entry's body is the measure.

### Board entry points

- Subtract hibernation from every elapsed and idle figure: `sleep.go` reads systemd-sleep's suspend and hibernate intervals from the journal, and `subtractSleeps` cuts them out of the run's and each role's working intervals.
  The span (`AwakeSpan`), idle (`Idle`), role time and each Producers row's elapsed and idle (`awake`) count awake time only, per run and in "All runs".
- Rank the share by active time: `ProducerRow.Active` is elapsed minus idle, printed as its own column; the share is active over the table's summed active time, and rows sort by active time.
- Map the old Burler role names onto the new ones: `roleOf` drops a trailing `-chair`, so `burler-review-chair` counts as `burler-review`, and `TestCountRun` sums both titles into one row.

Every point is met, and no change falls outside them.
The `-no-sleep` flag and the header line naming where sleeps came from support the first point: the header line answers the record's missing-journal bound, and the flag keeps the host's journal out of the integration test, as `PATTERN-test-speed` asks.
The package doc in `main.go` describes the new behaviour.

### Open risks

The record's two open risks are a machine powered off rather than suspended, and journal retention dropping a run's window.
Neither names a parent correction, and the record carries no review carry-over block.

### Overlap with peers and parent commits

`lyx loom peers` lists no parent commits.
No peer changes `tools/tokencount` or `test-budget.yaml`:

- `shed-tackler` (row 7) and `reed-launch-fixes` (row 0) list no changed paths, and their decision records do not mention tokencount or machine sleep.
- `twill-fork0` (row 9) changes stencils, `internal/burlerengine`, `internal/loomengine`, `internal/twilldirective` and `docs/overview.md`, none of which this run touches.

No landing order applies, since nothing overlaps; this run's own row is 22, further along than every peer.
The shared bookkeeping file `cmd/lyx/testdata/test-budget.yaml` changes only the `tools/tokencount` row.

### Superseded entries

This run makes no other board entry obsolete.
