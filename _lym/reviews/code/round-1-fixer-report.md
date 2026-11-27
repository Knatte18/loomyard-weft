---
dispositions:
  - id: F1
    disposition: deferred
    reason: "The premise holds: shedengine still persists state failed with the transient class. Building the entry's first point is a design tradeoff that belongs to the run's parent, as the review's own fix key says. It needs a parent ruling between the narrower reading (failed plus the transient class counts as the retryable stop) and a new retryable stop carried through every reader of the state."
  - id: F2
    disposition: fixed-differently
    reason: "The proposed wording gives the network waits and the tackler to the driver in general. Only the Go driver does that: contracts/stencils/shed/shed-template-driver.md:85 sends a transient stop of any class to one fork under the repair cap. The proposed bare loop re-step also holds only for a step loop. The three texts name both drivers and the step loop."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F2: reworded the `AnomalyTransientStop` doc comment in `internal/loomengine/anomaly.go`, the transient-stop paragraph in `internal/loomcli/doc.go` and the drop rule in `contracts/stencils/friction/friction-template-reflection.md`.
  Each now calls the halt an infrastructure failure classed transient that a re-step can clear.
  Fixed differently: the proposed "the loop re-steps it once, the driver also re-steps with waits, any other class goes to the tackler" was not written as is.
  The texts say a step loop re-steps it once, the Go driver does the network waits and the tackler hand-over, and the prompt-driven driver repairs under its repair cap.
  Two commits: the first made the proposed wording, the second corrected it after the self-check.
- F3: the `ensure` doc comment in `internal/landingshed/parentguard.go` now says a gap points to a wall-clock step, or to a host suspend where the monotonic clock pauses in suspend.
  A second commit put that sentence on one line, after the round's gate refused its wrapped form.
- F4: added the `TestClassifyHalt` row `FailedBuildSkewTransient_BuildSkew`.

Gate: `lym gate test` on `internal/loomengine`, `internal/loomcli`, `internal/landingshed` and `contracts/...` passed with no failures.
It passed again on the packages the self-check correction changed, and on `internal/landingshed` after the line-break repair.
Nothing was pushed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM (re-step wording is false for the prompt-driven driver): fixed with own change, naming both drivers.
- fix-consistency, LOW (`_lym/landing/summary.md` keeps the unqualified suspend claim): deferred, outside the write surface.
- fix-proposal, LOW (the loop re-step is stated unconditionally, and "the loop" is undefined): fixed as proposed, "A step loop re-steps it once".
- fix-proposal, LOW (the same suspend claim in `_lym/landing/summary.md`): merged with the fix-consistency one, deferred.
- fix-affected-set, LOW (the same suspend claim in `_lym/landing/summary.md`): merged with the fix-consistency one, deferred.

## Disputed

None.

## Deferred

- F1: needs the parent's ruling.
  The entry asks that a transient failure be recorded as a retryable stop, not as `failed`, at least until the driver's re-steps are spent.
  The branch keeps state `failed` and changes only the halt note's anomaly kind.
  The change description's `## Design decisions` argues for that, but the decision record holds no entry that allows it.
  The parent either records the narrower reading, which leaves the code as it is, or asks for the new retryable stop.
  That stop reaches `internal/shedengine/run.go`, `internal/sheddriver/driverules.go`, `internal/loomcli/halt.go`, the run state machine and the board status.
- Self-check, change description: `_lym/landing/summary.md` is a run record outside the fixer's write surface.
  Under `## Changes` it still says the two times show "whether the lock was held that long or the host was suspended".
  A gap between them only points to a suspend on some platforms, or to a wall-clock step.

## Rulings

None.
