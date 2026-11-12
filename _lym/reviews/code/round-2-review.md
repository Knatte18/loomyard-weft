---
verdict: APPROVED
---

No finding stands. Each focus question is settled below.

## Rejected

- Output of the upkeep pass (`internal/beamcli/dobbydeps.go`, `startTestMap`): stdout and stderr go to a `lymtemp` file, not a pipe the dobby reads. The pass holds its own descriptor, so it runs to completion after the dobby exits or re-execs. The leftover file in the dead dobby's run directory is swept by `lymtemp`. Not a defect.
- Anchor-move scan (`internal/fabriccli/anchordeps.go`): `TestLiveUnsparedChild` pins that a child registered as `RoleTestMap` is spared, and that a child registered under another role or nowhere counts. `dobbyPassInFlight` calls `liveUnsparedChild`, and the prime's owner is `runproc.HubOwner`, under which the pass registers, so `EndDobbies` through `EndSparing` ends it. Not a defect.
- Wording in `internal/orchcli` (`doc.go`, `stop.go`, `cli.go`, the beamdown error text) and in `runproc/record.go`, `docs/overview.md` and `pattern/PATTERN-test-speed.md`: all call the process the coverage-map upkeep pass that registers itself. No test-map refresh wording is left in them. Not a defect.
- State record (`internal/dobbyengine/state_test.go`): the row reads a record holding `landings_since_test_map`, asserts `PID` and the run's `FullPassOwed`, and asserts the key survives `UpdateState`. Not a defect.
- Reconcile walk (`internal/configsync/configsync_test.go`): the `TestReconcileAll` row seeds `test_map_every: 0` in beam and asserts the migration line, `test_map_upkeep: false` in the file, and no `test_map_every`. Not a defect.
- Start rule, config migration and layer hook (`internal/dobbyengine/testmap.go`, `config.go`, `internal/configengine/layers.go`): these match D8 and D11 of the decision record, including the once-per-episode exit log, the two-slot hold and the in-memory migration before `DropRetired`. Not a defect.
