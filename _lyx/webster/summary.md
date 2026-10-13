# Go driver per task, with the tackler and the arbiter behind it

All four batches (cards 1 to 22) recorded `status: done`, in order, with no recovery needed.

## Batch 01 — shed-paused-reasons (cards 1 to 5), head `fd0ccc90c`

Every `paused` step stop now carries a reason.
`driver_kind` and the three tackler keys are config keys open to the local layer.
`shedrun` gained the driver lock, tackler and settle paths, and `loomengine` the tackler and settle records.
`buildvcs.OfFile`, `LastStepBuildChanged` and the exported `RunUntilStop` and `RunGoto` bodies landed, with the recipe-blind tackler stencil.
Deviations: `internal/loomengine/driver_test.go`, `internal/loomcli/wiring.go`, `internal/shedverbs/steprecord.go`, `internal/shedrun/paths.go`, `internal/buildvcs/{buildvcs,doc,buildvcs_test}.go`, `internal/shedengine/run.go`.

## Batch 06 — arbiter-case-brief (cards 6 to 11), head `225c78c85`

The arbiter gained its case-brief marker and check-the-facts rule.
The tackler call (spawn, wait, verdict validation, paused-time deadline, tamper check) and its `resume` consume path landed.
`RecordMatchesHalt` is exported and `PublishRetrySpent` added.
The closed drive rule table, stop parse and hand-back (stop report, park, notify rules, watched marker) landed as pure functions.
Deviations: `internal/loomcli/arbiter_test.go`, `internal/battenshed/publishretry.go`, `internal/shedrun/doc.go`.

## Batch 12 — drive-loop (cards 12 to 16), head `324c1e967`

The settle loop, agent calls, restart reuse, hand-over to a fresh driver and the hidden `lyx loom drive` verb with its driver lock landed.
Start spawns `drive`, and resume spawns a Go driver for a halted driverless go run.
Deviations: `internal/loomcli/driverules.go`, new `internal/loomcli/drivewiring.go`, `internal/loomcli/smoke_test.go`.

## Batch 17 — loom-seed-driver-kind (cards 17 to 22), head `7c5ea3976`

Loom's seed writer resolves the driver from `driver_kind`, and batten records `child_driver` only when typed.
Batten reads the driver lock, holds its notice for a tackler, honors a Go verify retry and re-steps a halted driverless go child after a deploy.
The orch stencil says a go-driven run has no driver to message.
Deviations: new `internal/battenshed/innerrun_binaryrestep.go`, `internal/battenshed/{innerrun,deps,publishretry,notice,innerrun_halted_test}.go`, `internal/battencli/{cli,lifecycle_integration_test}.go`, `internal/loomcli/{wiring_seams_test,start}.go`, `docs/overview.md`.

## Verify gate fixes

The first verify attempt failed on four findings: a fabric-vocabulary leak in `internal/battencli/wire.go` and `internal/loomcli/drivewiring.go`, and three `internal/testkit/envkit` tests reporting five unfilled `InnerRun` seams.
The fixer's reply says it changed three things in one commit, `647ccecb5` (`fix: classify the new InnerRun seams as nil-legal and drop the fabric-vocabulary leak`):

- The doc comments in both files now say "task worktree" instead of the bare warp token.
- `internal/testkit/envkit/envkit.go` lists `InnerRun.ChildDriverLockHeld`, `TacklerInFlight`, `GoVerifyRetrySpent`, `ChildBuildChanged` and `ChildDirtyPaths` as nil-legal.
- The same five entries left `intentionallyNil` in `internal/loomcli/wiring_seams_test.go`, so `TestWire_EverySeamFilled` still passes.

The fixer ran `lyx gate test --tags integration` over the five affected packages and reports them green; it did not run the plan-level verify.
