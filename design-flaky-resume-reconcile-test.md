# Flaky integration test: loomcli TestLoomResumeReconcilesTheHubConfigBeforeArming

`flaky-resume-reconcile-test` · Note · bug, loom

Imported from [issue #492](https://github.com/Knatte18/loomyard/issues/492).

The webster integration-suite triage reported `internal/loomcli.TestLoomResumeReconcilesTheHubConfigBeforeArming` as flaky: it failed the first verify run and passed or vanished on rerun, with no change in between.

This is abnormal because a test that fails nondeterministically blocks verify gates, and the run in question halted blocked at Publish with "verify failed after merging parent branch" (anomaly `escalation-to-human`).
The halt note and the verify log on disk do not show whether this test caused that failure, so the link is unconfirmed.

What to do: reproduce with repeated runs of `lyx gate test --tags integration ./internal/loomcli`, find the ordering, timing or shared-state dependency in the test, and make it deterministic.

Provenance: task `bolt-and-store`; notes `webster-verify-triage.md`, `loom-halt.md`.
