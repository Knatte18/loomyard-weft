# Resume lost gate strands; reconcile runs an interrupted step left

Two bugs where a leftover strand or shuttle run stopped a run.

Closes #143
Closes #138

## Gate re-prompt to a strand reed lost (#143)

A gated run whose writer had to be re-prompted halted when the writer's strand could take no keys.
That happens when reed no longer tracks the strand, holds no pane for it, or its pane died.
`liveStrand` now wraps each of these failures in `errStrandGone`.
`handleGatedBoundary` catches it and calls the new `relaunchForReprompt` (`internal/shuttleengine/relaunch.go`).
That resumes the writer's session in a fresh strand, with the re-prompt as the resumed session's first turn.

- The run keeps its run directory, events file and gate state. Only the strand changes.
- The record names the new strand before the old one is removed.
- The new provider's startup reads the events file only past its size at the resume,
  so the first session's lines never mark the resumed session as started.
- The resume needs an engine that can resume a session and name the process holding one (`SessionHolderFinder`).
  It is refused when a live holder is found or cannot be ruled out,
  so a writer still working in a pane reed lost is never run twice.
- A refused or failed resume ends the gate loop and the reason names both failures,
  so the halt is immediate and no longer follows a long stall.
- The resumed session keeps the run's skills for its settings and passages but takes no skill-load turn.

## Re-step after an interrupted stop (#138)

A re-step after an `interrupted-stop` ended on a shuttle run the interrupted step left reading `running`.
Two changes remove that leftover:

- `Runner.StopStepRuns` now decides per run what to stop.
  A running run recording the step is stopped when its strand is live or removable,
  and a run with a dead, unremovable strand has only its record stopped.
  A run whose strand reed does not track, or tracks without a pane binding, may still have a live agent.
  It is stopped only when its engine can name the session's holder and finds none.
  Otherwise it stays running and is listed under the new `StepStopReport.Held`.
  A run whose output files all exist is left running and listed under `StepStopReport.Finished`.
- The shed loop gains `LoopSpec.OnStepOrphaned`, which gets the trace id of a step whose process is gone without an envelope.
  It runs for a child that exited without an envelope and was not killed by the watchdog,
  for the step a panic or a dead loop leaves in flight,
  and once before the first child when the loop starts on a failed status naming an interrupted step.
  In that last case the step is the newest interrupted record in the steps directory.
  The note it returns ends the interrupted cause, or goes to the loop log.
  `internal/loomcli/looparm.go` wires the hook to `stepRunsNote`, which renders what the stop did.

The interrupted cause now names what was done with the leftover runs.
That answers the request to record the cause where the loop exits.

## Decisions to review

- A strand reed cannot address is never assumed dead.
  `StopStepRuns` and the resume both stay conservative and leave a run in place when a live holder cannot be ruled out.
- Reconciling before the first child reads the newest interrupted record by modification time, not the step named in the status.

## To check by hand

- `leftover-runs-not-reconciled-on-restep` (MEDIUM) was left open by the last code review.
  Check `reconcileInterruptedStep` in `internal/shedverbs/loop.go`:
  it recognizes an interrupted stop from the status error text, so a leftover run from a step with no interrupted record is not reconciled on a re-step.
- Neither fix was exercised against a live tmux and reed session.
  Reproduce by killing a writer's reed strand mid-gate, and by interrupting a step and re-running `lym shed step --until-stop`.
