# Round 1 fixer report

Verdict was APPROVED; all three findings were fixed, each in its own commit, nothing pushed.

## Fixed

- **F1** (LOW, scope): `TestObserveEntry_ConsumesHandoffVoucherIntoObservation` in `internal/loomcli/handoffvoucher_test.go` now seeds a seven-row history and asserts `RecentHistory` holds the last `recentHistoryRows` rows.
  No new test function; the coverage gap closed inside the existing test.
- **F2** (LOW, consistency): the sentences this diff added or rewrote in `LoomHandoffVoucher` (`internal/loomengine/config.go`), `handoffVoucher` and `recordHandoffVoucher` (`internal/loomcli/entryobservation.go`), `EntryObservation.Vouched`, `RecentHistory` and `DetectCrashResume` (`internal/loomengine/anomaly.go`) and the "Why the leaf exists" paragraph (`internal/friction/doc.go`) now use one sentence per line.
  Content unchanged; untouched legacy lines were left as they were.
- **F3** (NIT, consistency): the three `observeEntry` warnings say "entry observation" instead of "self-report entry observation".

## Verification

`go build ./...` and `go test` over `internal/loomcli`, `internal/loomengine`, `internal/friction` and `cmd/lyx` pass.

## Deferred

Nothing.
