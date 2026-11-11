# lym machine name shows and sets the machine name and sweeps owners across a rename

Batch 01-owner-replace-set recorded `done` at head `4b63dcd6573c7abf5e46a2dcad87806af1ca0de1`.
The batch held four cards, committed in order: the owner write's replace set in `boardengine`, `machinename.FromThisHost`, the `machine` module's `lym machine name` show verb, and the set verb that sweeps hubs.

## Digest

- Renames: none reported.
- Departures: none reported.
- Deviations (scope guard): the files under `internal/machinecli/`, namely `cli.go`, `cli_test.go`, `current.go`, `set.go`, `set_test.go` and `sweep_integration_test.go`.
  They are the new module's own files, created by cards 3 and 4, and the digest lists them only because they sit outside the batch's declared target list.
- Warnings: none.


## Integration suite triage

The `final` gate failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/gatecli.TestGateTest_DetachedRun/a_passed_lease_ends_a_--wait_gate_line_through_its_child,_which_kills_its_tiers_and_frees_its_slots`
- `github.com/Knatte18/loomyard/internal/gatecli.TestRedundancyStep/ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged`
