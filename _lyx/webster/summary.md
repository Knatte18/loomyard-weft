# Orch context line in the batten overview

Batch 01 (`01-orch-state-peek`) recorded `done` at head `0e1a4465dfb254e5c693e89855dbeb60619db189`.
It added `orchengine.PeekState`, which reads the orch state file with no lock and no mkdir.
It also added the overview's orch line above the run lines, with thresholds loaded once at start.

The scope guard listed three deviations from the batch's declared targets: `internal/battencli/overview_test.go`, `internal/battencli/overvieworch.go` and `internal/orchengine/state_test.go`.
`record-batch` returned no warnings.
