# Card 26 — loom-cli-refusal-sweep

**Edit:**
- `internal/loomcli/approve.go`
- `internal/loomcli/start.go`
- `internal/loomcli/commitrecords.go`
- `internal/loomcli/approve_test.go`
- `internal/loomcli/start_driver_test.go`
- `internal/loomcli/commitrecords_test.go`

**Intent:**
Give every refusal of loom's own hand-written verbs a way forward (D1) and a reaching test, in the trailing `way forward: …` form.
Loom's gate rows are card 27's.

Messages to fix:

- `approve`, run not awaiting or blocked at Publish: `lyx loom status` shows where the run is; approve once it halts at Publish.
- `approve`, no pull request, or pull request not open: step the run to Publish (`lyx loom step`), which opens it.
- `start`, "driver did not take the run lock": re-run `lyx loom start`; the named driver log says why the driver exited.
- `commit-records`, commit failure: transient; re-run `lyx loom commit-records`.
  The ly-drive driver runs this verb at every stop and park, so the failure is reachable on every run.
- `commit-records`, "the commit landed locally but was not pushed": `lyx fabric push` pushes the landed commit, or re-run `lyx loom commit-records`.

Already carrying a way forward, test-only where untested: approve's missing status file and its HEAD-differs refusal, start's not-parked refusal, loom's step busy refusal, and the missing-seed refusal.

Grouped: approve's branch-resolution, origin-URL, pull-request lookup and task-HEAD read failures, and commit-records' merge-state probe and park-marker write failures, all raw I/O (transient: re-run the verb).
The `validate-discussion`, `validate-plan` and `validate-description` "not yet valid" envelopes are each verb's verdict on its artifact, not a refusal of a run, and get no row.

Tests: each fixed row is reached from a realistic state, the message asserted to contain its way forward, the way forward taken, and the refused verb asserted to succeed.
The approve rows use a `hubforge` hub; the commit-records rows drive `commitRecordsVerb` through its injected `commitStatusDeps`, failing the commit or the push once and succeeding on the re-run.
Tests that spawn git carry the `integration` tag and use `internal/hubforge`.

**ImpactSummary:** Message text plus tests in loom's approve, start and commit-records verbs; envelope keys are unchanged, and anomaly and ly-drive classification read state and kind, never these texts.
