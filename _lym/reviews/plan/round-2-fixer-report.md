---
dispositions:
  - id: F18
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- F18, fixed as proposed, in `_lym/plan/14-goto-clears-pr-review.md`.
  The `prReviewGotoAdmitted` bullet no longer stops through `sessionLiveness(c.reed.Status)` and `c.runner.StopStrand`.
  `stopLiveSession` now calls `StopStrand` on the runner `c.stepStopRunner()` returns, resolved inside the closure when it is called.
  The bullet says why: `goto` runs under `wireLightweight`, which leaves `c.reed` and `c.runner` nil.
  The bullet was reordered so the `commit` clause stays in the list of seams, ahead of the explanation.
  Under **Uses:**, `internal/loomcli#sessionLiveness` became `internal/loomcli#loomCLI.stepStopRunner`.
  `lym loom validate-plan` passes and reports only the baseline's informational findings.
- Self-check fixes, in the same bullet:
  - The sentence "`StopStrand` does nothing for a strand reed does not track" now states what `StopStrand` does.
    It marks a run that still reads running as stopped.
    It removes a strand only when reed tracks it as removable: live, held, or an own-window strand with a dead pane.
    The bullet also says why the wider stop is wanted.
  - The bullet says that this path loads the reed and shuttle configs.
    A load error is a stop error, so the record keeps its fields.
    The hook's text names the failed load, which the operator repairs before running `goto` again.

## Self-check

- fix-consistency: no findings.
- fix-claims, MEDIUM: the sentence "`StopStrand` does nothing for a strand reed does not track" leaves out the write to the run record and the wider removal.
  Fixed as proposed.
- fix-proposal, MEDIUM: on the lightweight `goto`, the runner `stepStopRunner` builds depends on module config, and the card gave no way forward when that load fails.
  Fixed as proposed: the lens named no concrete change, so the card now names the failed load as the thing to repair before running `goto` again.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
