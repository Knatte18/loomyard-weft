# Coverage map uses: reach note, failure triage and changed-line mutation

All eight batches (cards 1 to 13) recorded `done`. No batch reported a rename or a `Pattern-Departure:` trailer.
Every digest's `deviations` list is informational; no batch went stuck or needed recovery.

## Batches

- **01 (cards 1-4), head `7b7e5828`:** opened `internal/testreach` with the changed-function finder and the cross-package map reader.
  Added `PATTERN-coverage-map-reader` with its background and guard test, and `gate.yaml` accepting `reach` and the three new keys.
  Deviations: the new `testreach` files and the `cmd/lym` guard test, all inside the batch's own work.
- **05 (cards 5-6), head `1ac6a287`:** the `reach` report step writing `reach.json` and the note, and the stamped index that only the prime's `lym gate test --map` writes.
  Deviations: `gatecli` reach and map files and their tests, and the `cmd/lym` guard.
- **07 (card 7), head `e5def9f9`:** the `reach.json` reader, its HEAD match and the code review rubric's `reach_report` marker.
  Deviations: `reachreport.go`, `loomcli/reachnote.go`, `shedadapters/rubric.go` and the rubric test.
- **08 (card 8), head `1af7f0e2`:** `GateResult.WriterOnly` bypasses the cold fixer, and `RerunMark` marks its post-fixer re-run.
  No deviations.
- **09 (card 9), head `4b9f7325`:** the reach turn for the loomshed verify gate and the weaver batch gate.
  The first fork was stopped by the operator and then resumed by them; the Master's re-fork was stopped, so one fork's work counts.
  `record-batch` warned that two fork transcripts counted toward the batch.
  Deviations: `reachreport.go`, `rowgate_test.go`, `entries_gate.go`, `verifytree/reach.go` and its test, and weaverengine test support.
- **10 (card 10), head `6b72be09`:** `lym gate reach` for one function and over a red parent branch's landings.
  Deviations: `parenttips.go`, the reach verb and its test, and a fix in `testreach/index.go` so a read-only verb leaves no lock file.
- **11 (cards 11-12), head `81d86707`:** triage lines in the failure report and in every fixer's report.
  The prime follows the parent decision: no parent branch, so the no-change-set line and no D8 command.
  Deviations: gatecli counters, failure report and triage files, `gateslot/triage.go`, and weaverengine test files.
- **13 (card 13), head `a54e624b`:** the changed-line mutant planner, `lym gate mutate` and the trial's board note.
  The `mutation` report step is not built, per the decision record.
  Deviations: the `gatecli` mutate files, `redundantmutants.go`, `testcover` line-mutant and mutant files, and `testreach/changedlines.go` with its test.

## Scope guard

The scope guard listed only files tied to each batch's own work, with no unexpected targets.
Each fork reported its card and batch gates passing, with tier2 and tier3 not run.
