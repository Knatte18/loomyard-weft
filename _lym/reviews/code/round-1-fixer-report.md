---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed told flag set inside the closure cannot work: a closure cannot tell a re-check after a failed send from the new arrival that follows a delivered one, so resending on every call would loop the writer through turns. The loop now reports delivery through a new GateResult.Delivered *bool the closure reads, so the hold line is offered until the send lands."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The proposed test needs a newer-stencil build error from the real table build, which the loomcli tests cannot make: the skew state is process-global in stencilstore and set only through its own export_test.go. PreflightStencils now builds through c.env.DarnTable, so the wiring test overrides that seam to return the wrapped NewerBoardCopyError and drives PreflightStencils itself. The helper-only test is replaced."
---

# Fixer-report, round 1

## Changes

Three commits, grouped by shared files, plus one for the self-check:

- `f3a9a365`, F1, F2, F4 and F6 (all in `internal/loomshed/redfirstgate*.go`):
  - F1: `redFirstHoldSendFailed` is built from `gateslot.NoTestSeamTitle`.
  - F2: the hold keeps offering its line on every pending answer until the loop reports it delivered.
    `shuttleengine.GateResult` gained `Delivered *bool`, which `evaluateGate` carries and `handleGatedBoundary` sets after a send that landed.
    `redFirstGate` holds `holdLine` and `told`, resets `told` when a new hold starts, and refreshes the line while it is untold.
    `TestGatePending_FailedSendWarnsAndStaysPending` asserts the flag; the hold test fails the first send and sees the line offered again.
  - F4: `redFirstGate` keeps the held refusal's `verifytree.Result`; `final` returns it as a failure without calling verify when HEAD's tree is the held tree.
    `TestRedFirstGate_FinalOnTheHeldTreeFailsFromTheHoldWithoutRunningTheCheck` pins one verify log.
  - F6: the hold test re-arms after a passing run (the command's exit code is read from a file) and then lifts on the record.
    Two table rows cover a timed-out exit 4 and exit 5 failing terminally, and two cover `Final` passing an exempt entry without a run.
- `e0b12a31`, F3, F5 and F7:
  - F3: `latestDarnOutcome` names the kept findings file only when it was written no earlier than the latest Stuck's `StartedAt`.
    Two `TestLatestDarnOutcome` rows pin both sides.
  - F5: the four stale comments (`GateFindingsFile`, `gateFindingsPath`, the `doc.go` prior-work paragraph, `DarnOutcome.FindingsPath`) state the keep contract.
  - F7: `PreflightStencils` builds through `c.env.DarnTable`; a `TestWire_DarnRecipe` subtest drives it with a wrapped `*stencilstore.NewerBoardCopyError` and with another error.
    `TestPreflightTableError_KeepsANewerStencilRefusalsOwnWayForward` is removed, since the subtest covers the same two rows through the wiring.
- `c5e9eb87`, self-check findings (see below).
- A fourth commit answers the comment gate's findings: five comments the round added broke at a fixed column or ran too long.
  They are in `redfirstgate.go`, `redfirstgate_test.go` (two), `shuttleengine/run.go` and `shuttleengine/wait.go`, and now read one sentence per line.
  It changes comment text only.

Gate: `lym gate test --tier 0,1` over every package that uses `GateResult`, plus `shedrun` and `shedadapters`, passed with no failures.

## Self-check

- fix-code-correctness, BLOCKING: a `Sent func()` field made `GateResult` non-comparable and broke `internal/verifytree/reach_test.go:130` and `internal/gatefixer/rowgate_test.go:335`. Fixed with my own change: a `Delivered *bool` field keeps the struct comparable and needs no test rewrites (commit `c5e9eb87`).
- fix-consistency, LOW: `wait.go` doc line over the limit. Fixed (`c5e9eb87`).
- fix-consistency, LOW: `loomshed/doc.go` prior-work line at 120 characters. Fixed (`c5e9eb87`).
- fix-consistency, LOW: `redfirstgate_test.go` doc comment first line at 128 characters. Fixed (`c5e9eb87`).
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
