# Webster-Burler round 5 — fixer report

Every finding was validated against the code and fixed, one commit each.

## Fixed

- **F1** (`c1596c8b5`) — tests now reach the gate.yaml way forward at each Go-side gate site.
  - `internal/hubgeom/webstergeom_test.go`: the absent-file check also requires `run "lyx fabric reconcile"`, and a new step writes `slots: 0` and requires `fix it with "lyx config gate" from the prime`.
  - `internal/websterengine/cardverify_test.go`: the failed-acquire row loads a real `gate.yaml` holding `slots: 0` through `gateslot.LoadConfig` and pins the full failing card line, way forward included.
  - `internal/verifytree/verify_integration_test.go`: `TestVerify_SlotGate` ends with a `Verify` over a pool whose `gate.yaml` holds `slots: 0`, asserting the returned error names `lyx config gate`, the command never ran and no marker is left.
    It is a step of the existing test because the package sits at its test budget.
- **F2** (`288b45af2`) — decision: keep the degrade.
  A slot the plan gate's type load cannot acquire is a failed load, answered by the import-path scan, the contract a failed or timed-out `go list` already has.
  The plan gate is not the run's last line: the next slotted gate (the verify gate, Publish, Finalize) refuses on the same `gate.yaml` with its way forward.
  `internal/planglyph/doc.go` and `contracts/specs/loom-plan-spec.md` (`caller-uncovered`) state it, and `TestGlyphChain_CallerCoverage` gains a row that validates through `NewSlottedIndex` over an unusable `gate.yaml`, pinning the scan's findings, no error and no wait record left.
  The row spawns no process, so it sits in the untagged tier.
- **F3** (`d27f01325`) — new kit `internal/testkit/indexkit` with `ValidateStub`, a `planindex.Index` whose `Validate` answers one fixed finding.
  It is a kit because both CLI packages use it (`PATTERN-testkit`).
  `TestValidateCmd_ScopeFollowsRunProgress` gains two rows, no run and a run with no batch begun, covering both `c.geom.Index.Validate` sites in `scopedValidate`.
  `TestValidatePlanCmd` gains a row in which `--require-approved` must report the stub's finding, while flag-absent mode still runs the real index.
  A revert of any of the three sites to `planglyph.Validate` now fails a test.
- **F4** (`cd55eaece`) — the `plan gate pass` slotted row is replaced by one that runs the index's `ValidateFormat`, the plan gate's own entry, so the two slotted rows cover distinct public entry points.

## Verification

The hub has no `gate.yaml`, so `lyx gate test` refuses (`gate config absent; run "lyx fabric reconcile"`), and creating that file would mutate the hub outside this worktree.
I ran plain per-package `go test` instead, and all of it passed: untagged `cmd/lyx`, `internal/pattern`, `internal/testkit`, `internal/loomcli`, `internal/webstercli`, `internal/planglyph`, `contracts/specs`; integration-tagged `internal/hubgeom`, `internal/verifytree`, `internal/planglyph`, and `TestRerunCardVerifies*` in `internal/websterengine`.
`lyx loom lint-comments` (dev build) is clean.
The full module-wide tiers were not run.

## Disputed

None.

## Deferred

None.
