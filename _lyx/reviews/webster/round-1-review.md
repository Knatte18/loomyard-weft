---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shuttleengine/wait.go:1496
    summary: Final keeping precedence over Forked in evaluateGate has no test
    fix: Add one case to TestGateList_FinalReports in internal/shuttleengine/gate_list_test.go, or a row to TestGatePending_MidTurnTicksNeverReevaluate_DeadlineUsesFinal in gate_pending_test.go. The case uses a MayHold or PassOnCap entry that sets Gate, Forked and Final, over a run whose RunState has ForkSubagents true. At a final evaluation it asserts that only the Final closure was called, so swapping the order of the Forked and Final assignments in Run.evaluateGate fails the test.
---

Range: `status.json` gives `product.parent` as `main` and no rework round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, seven commits, one per card.
The tests of the touched packages pass under `lyx gate test`: `internal/shuttleengine/...`, `internal/loomshed`, `internal/shedrecipe`, `internal/loomcli`, `internal/twilldirective`, `internal/loomengine` and `contracts/stencils`.

### [LOW] Final's precedence over Forked is untested

**Location:** `internal/shuttleengine/wait.go:1496`
**Issue:** `Run.evaluateGate` picks `entry.Gate`, then `entry.Forked` for a fork-authorized record, then `entry.Final` at a final evaluation.
The diff and the doc comments state that `Final` keeps precedence over `Forked`, but no test sets `Forked` and `Final` on one entry.
`TestAttach_GateThreading` pins the Forked-versus-Gate choice and the fork audit, and the existing `Final` tests never set `Forked`.
Reordering the two assignments would pass the whole suite.

## Settled focus questions

- Cards 1-2, zero value: `TurnProgressUnknown` is the iota zero.
  `holds` consults the veto only through a `TurnProgressReader` type assertion, so an engine or fake without the capability falls to the existing idle timer.
  `unmatchedTurnStart` sets `restarted` on a session start after the pending turn start, and `holds` skips the veto when `restarted` is true.
  The rows in `TestSend_WaitsForIdleSession`, `TestGate_WaitsForIdleWriter` and `TestRunner_SessionIdle_ReturnsScriptedClassification` cover both.
  The order is as the card says: released stays released, a reported interrupt, the veto, then the pane timer.
  `boundaryIdle` ignores `restarted` correctly, since it only clears the boundary.
- Card 2, reader: `newestTurnProgress` drops the unterminated tail, then scans backward past blank, unparsable, sidechain and non-user/assistant lines.
  It classifies an assistant entry by `stop_reason` and treats the interrupt marker as unknown.
  The index arithmetic is correct at both ends of the data.
  The fixtures each pin a different shape, and the rows over prompt, task notification and tool result differ in transcript shape, as the card specifies.
  Not a defect.
- Card 3: `RunState.ForkSubagents` is written once in `Runner.start` and read in `finalize` and `evaluateGate`.
  It is `omitempty`, so an older record decodes false and behaves as before.
  No other production site reads `spec.ForkSubagents` to decide the audit or the closure.
- Cards 4-6: only `planWriteEntry` sets `Forked`, through `planWriteGateSpec`, and `resolveGateSpec` has no row-name switch.
  `NewLensReviewPlanGate` and `validate-plan --lens-review` both call `ValidatePlanLensReview`.
  The flag refusals run before the plan is parsed and carry a way forward, and the refusal-spec rows and `PATTERN-gate-self-check-parity` are updated in the same commit.
  The card's Rename group is satisfied: the old test name no longer appears anywhere.
- Card 7: step 8 of the directive uses `final_self_check_command`, the Plan host passes `lyx loom validate-plan --lens-review`, and the Discussion host passes its unchanged command.
  Step 3 keeps `self_check_command`.
- Test economy: the only new top-level test is `TestTurnStartProgress`, with its budget row raised by one.
  Every other test is a row or subtest in an existing test, as the cards direct.
  The one gap is F1.
