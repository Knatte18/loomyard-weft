# Plan round 1 — fixer report

## Changes

- **F1** (fixed as proposed): premise confirmed at `internal/loomcli/arm.go`, where `resolveRunID` takes `args[0]` as the run-id for every verb but `reject`, `insert-card` and `remove-card`, and `resolveRecipe` passes it to `shedrun.ReadSeed`.
  Card 6 now lists `internal/loomcli#loomCLI.resolveRunID` and `internal/loomcli#TestResolveRunID` under **Edit:**.
  Its Intent says `resolveRunID` adds `fmt` to the verbs whose positional arguments are not a run-id, and that the check's comment names fmt's file arguments.
  Its Tests gain the `TestResolveRunID` row for `fmt` with `internal/x/y.go` and `wantRunID: shedrun.SelfRunID`, noting `TestFmtCmd` skips the pre-run.
  The ImpactSummary names the `resolveRunID` change.
- **F2** (fixed as proposed): card 6's `planGoFormat` paragraph adds a nonexistent path to the refused reasons, as `<path as named>: no such file` from the `filepath.EvalSymlinks` failure before the inside-root check.
  `TestFmtCmd` gains a row where a nonexistent `.go` path beside a misformatted file refuses the whole call, `refused` naming the missing path, the misformatted file left unwritten.
- **F3** (fixed as proposed): premise confirmed in `cmd/lyx/destructiveguard_test.go`'s table.
  Card 7 lists `cmd/lyx#destructiveGuardRecordingExecutors` under **Edit:**, and its Intent says the table gains `{"removeWorktreeLeftover", "func removeWorktreeLeftover(rec *Mutations, "}` so `TestMutationRecord_FabricengineProductionSource` pins its leading recorder.

`lyx loom validate-plan` passes after the changes (`ok: true`, informational findings only).

## Disputed

None.

## Deferred

None.
