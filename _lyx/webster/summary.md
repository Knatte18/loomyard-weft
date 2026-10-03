# Tier 1 anomaly filing: pending retry, step coverage, halt-filing ownership

Batch 10 (`anomaly-pending-retry`, `c20ebe2f`): a failed Tier 1 filing is now kept in the filed-title marker as a pending title plus body and retried on every later detection pass.
`detectAndFileAnomalies` reaches the filing pass past its skips, and the status read moved into a new helper, `detectFinalAnomalies`.
The `LoomSelfreportFiled` doc and the tests in `selfreport_test.go` and `selfreport_github_test.go` were updated.
No deviations from the declared file-ops.

Batch 11 (`anomalies-under-step`, `afcee51b`): `loomCLI` gained a `fileIssue` seam, wired to `selfreportengine.CreateIssue` in production and to a refusing filer under `testing.Testing()`.
`loomAfterStep` runs `detectAndFileAnomalies` first on every path, and the entry observation is taken after the busy probe via `observeStepEntry`.
The halt note carries a Tier 1 line when the `selfreport` knob is on.
Changes are in `cli.go`, `arm.go`, `halt.go`, `halt_test.go` and `step_test.go`, within the card's declared files.

Batch 12 (`halt-filing-ownership`, `2ee701fc`): the reflection stencil, `CONSTRAINTS.md` (Friction Capture Invariant) and `docs/overview.md` state that Tier 1 files the halt event and the reflection files the lyx problems behind it.
A claim on the new stencil sentence was added to `claims_test.go`.
No deviations from the declared file-ops.

The batch records carried only informational scope-outside-plan warnings for symbols inside the touched files.
The integration suite (`## verify:`) reported `status: OK` at `2ee701fc`.
