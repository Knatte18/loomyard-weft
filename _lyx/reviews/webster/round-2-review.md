---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/boltpullfirst_integration_test.go:182
    summary: Test files the diff adds or edits in fabricengine and stencilstore still capture logger output with their own sink instead of internal/testkit/logcapture, against PATTERN-testkit
  - id: F2
    severity: LOW
    class: scope
    location: internal/fabricengine/boltpullfirst_integration_test.go:409
    summary: The no-seed-commits-ahead conflict row of the PushRecorded test asserts error, tip and empty record but not a clean tree or no rebase in progress
---

### [BLOCKING] Log capture is still hand-rolled in test files the diff touches

**Location:** `internal/fabricengine/boltpullfirst_integration_test.go:182`, `internal/fabricengine/housekeeping_integration_test.go:24` and `:124`, `internal/stencilstore/reconcile_test.go:482`

**Issue:** `PATTERN-testkit` says shared test support is shared through one kit under `internal/testkit/<kit>/` and never duplicated per package.
`internal/testkit/logcapture` is that kit for `logger` output (`Capture`, `CaptureVerbose`, a concurrency-safe `Buffer` and a `t.Cleanup` that restores `os.Stderr` and verbosity).
Round 1's F1 moved `cmd/lyx/stencilseed_integration_test.go` and `internal/hubreconcile/ensure_integration_test.go` onto it, and both now use `logcapture.Capture` with no local sink left.
The same duplication remains in three other test files the diff adds to or edits:

- `boltpullfirst_integration_test.go` is new in this diff.
  The `non-seed commits over a moved upstream skip diverged and warn` subtest declares `var logs lockedBuffer`, calls `logger.SetOutput(&logs)` and restores it by hand in `t.Cleanup`, which is `logcapture.Capture(t)` written out.
- `housekeeping_integration_test.go` keeps the package's own `lockedBuffer` type, a second copy of `logcapture.Buffer`.
  The diff edits `TestStoreHousekeeping`'s capture to add `logger.SetVerbosity(1)` and its hand-written restore, which is `logcapture.CaptureVerbose(t)`.
  `lockedBuffer` is used only by these two files, so it goes away once both switch.
- `internal/stencilstore/reconcile_test.go` (around line 482) gains `logger.SetOutput(&buf)`, `logger.SetVerbosity(1)` and a two-line restore around a plain `bytes.Buffer`, again `logcapture.CaptureVerbose(t)`.

`logcapture` imports only `internal/logger`, so no import cycle bars any of the three packages from it.
The packages' own test files such as `internal/stencilcli/cli_integration_test.go` and `internal/fabricengine/add_rollback_adopt_test.go` also carry the older pattern, but the diff does not touch their capture lines, so they are context and not the subject.

**Fix:** Replace the three capture sites with `logcapture.Capture(t)` or `logcapture.CaptureVerbose(t)` and read `buf.String()` as before.
Delete `lockedBuffer` and its `sync` and `bytes` imports from `housekeeping_integration_test.go`.
`TestStoreHousekeeping` is not parallel and `boltpullfirst`'s subtest comment already accepts the shared sink, so the switch changes no concurrency behaviour.

### [LOW] The no-seed conflict row does not pin the clean tree

**Location:** `internal/fabricengine/boltpullfirst_integration_test.go:409`

**Issue:** `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` has four rows.
The row `a retry that also fails returns its error and leaves the dropped board clean` asserts a clean status and no `rebase-merge` or `rebase-apply` directory.
The row `a conflict with no seed commits ahead returns the error without a drop` asserts the `ErrPullRebaseFailed` error, the unchanged tip and an empty record, and stops there.
`a conflicting board commit leaves the board at its tip…` has the same three assertions.
`PushRecorded`'s contract for these two paths is that the original error comes back with the board as it was, and a rebase left half-applied would pass the tip assertion while breaking that contract.

**Fix:** Add the same clean-status and no-rebase-in-progress assertions to these two rows, sharing the check with the retry row through a small helper in the test file.

## Focus questions settled with no finding

- **Log capture in `stencilseed_integration_test.go` and `ensure_integration_test.go`:** both capture through `logcapture`, and neither declares a local sink or capture helper.
  The remaining duplication is in other packages and is F1.
- **`lyx fabric reconcile` failure tests:** `TestRunCLI_ReconcileBoardFailureFailsTheVerb` covers a hub-config commit failure (rejected pre-commit hook), a hub-config push failure (rejected pre-receive hook), a warp-binding commit failure, a warp-binding push failure on the recorded path (unreachable remote) and one on the present path (rejected push).
  Each row asserts a non-zero exit, the `pairs` key kept in the envelope and the `hub_config_detail` or `warp_binding_detail` text.
  `TestRunCLI_HubMutationScenario` adds the skipped-heal row.
  Not a defect.
- **`reconcileHubWide` and the build stamp:** `reconcileHubWide` pushes on every walk and reports skipped on a push failure, and `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` reruns with nothing new to commit and asserts the stamp stays absent, then asserts it again over a moved upstream.
  `hubreconcile/doc.go` states the same guarantee, including the repeated divergence Warn.
  Not a defect.
- **`smoke_switchseed_test.go` comment:** it now says the dev channel stamp makes `stencilstore.ModeFor` return `ModeDev`, which warns instead of refreshing an untouched stencil.
  That matches the three-way `ModeFor(dev, production, cleanStamp)`, where a dev build without a clean production stamp maps to `ModeDev`.
  Not a defect.
- **Range and gates:** no rework round exists, so the range is `git merge-base main HEAD..HEAD` (parent `main`).
  I ran the integration-tagged tests of the changed packages (`cmd/lyx`, `buildinfo`, `buildvcs`, `cliwire`, `fabriccli`, `fabricengine`, `gitrepo`, `hubreconcile`, `pairteardown`, `reedcli`, `reedengine`, `stencilcli`, `stencilstore`, `testkit`, `tools/deploy`) with `lyx gate test`; all passed.
  `revisionLabel` and `shortRevisionLength` no longer exist in `hubreconcile`, and no caller of either remains.
