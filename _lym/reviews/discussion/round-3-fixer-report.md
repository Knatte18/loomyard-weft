---
dispositions:
  - id: F26
    disposition: fixed
  - id: F27
    disposition: fixed-differently
    reason: "Step 4's write leaves the copy fact out, since step 3 already writes it into the run directory before reed acts (D6 keeps it beside the launched-account record, not in the run record); the write also clears the record's started flag, which `markStarted` (internal/shuttleengine/wait.go:1564) persisted for the limited launch and an attach reads to skip its startup probe."
  - id: F28
    disposition: fixed
---

## Changes

All in `_lym/discussion/decision-record.md` unless named.

- F26: D7 step 4's startup-gate sentence is replaced.
  The relaunch's startup wait decides on the pane's capture alone and runs with the events proof off, as `Wait`'s probe of a not-yet-started run does, because the kept events file already holds the limited session's session-start and turn-start lines and `turnStartedAfterSessionStart` would short-circuit.
  A stale session-start line satisfying the line half is accepted by `checkLivenessTick`.
  A pane that never reaches ready ends the run as the startup wait ends it.
  New D7 criterion: a trust dialog on the relaunched pane is dismissed, and a pane that never reaches ready ends the run.
  The self-check narrowed the review's "trust or login dialog" to a trust gate and added the timeout outcome.
- F27 (fixed differently, see frontmatter): step 4 now writes the session id and `PromptLine`, clears the started flag and sets the prompt offset to the events file's size in one hold of the record's lock, and only then has reed relaunch the pane.
  A reed failure falls under the existing died rule for steps 2 to 4.
  The bound now also covers a live strand: past it, an attach or `Wait` clears the mark and attaches, and `DeliverPendingPrompt` types a `PromptLine` the dead `Wait` never sent.
  New D7 criterion: an attach after a kill between reed's relaunch and the mark's clear attaches with the new session id and offset and does not re-read the limited turn end.
- F28: the record-writes paragraph now states one rule.
  Every reader that tests for a running run reads a relaunching record as a live running run within its bound, through one predicate on the record.
  Past the bound, until an attach or `Wait` applies the bound, the predicate reads the record as a running run whose waiter is gone.
  Activity and session-state readers report a relaunching run with no turn-end facts.
  `Runner.StopStepRuns` stops a relaunching run whatever its pane state.
  The PATTERN bullet carries the same rule, shortened under the gate's 280-character cap for a rule text, and the plan-writer notes name the sites as the comparisons to `runOutcomeRunning` in `internal/shuttleengine`.
  New D7 criterion: a step stop during a relaunch leaves the run stopped with no strand up, carry-over does not treat the run as unfinished, and the activity read reports no usage limit.
- Step 1's live-reading sentence carries the bound, and the old kill-window criterion now covers only kills before reed's relaunch.
- `_lym/discussion/support-log.md`: Round 3 entry under Review rounds.

## Self-check

- fix-consistency, LOW, old kill-window criterion and closing bound sentence overlap the new live-strand case: fixed (criterion narrowed to before reed's relaunch; closing sentence says anywhere in steps 1 to 4).
- fix-consistency, LOW, step 1 dropped the bound qualifier: fixed.
- fix-claims, MEDIUM, the start path dismisses no login dialog: fixed (now "a trust gate").
- fix-claims, LOW, nothing says how other readers treat a record past its bound: fixed (merged with fix-proposal's fourth finding; past the bound the predicate reads a running run whose waiter is gone).
- fix-proposal, MEDIUM, a fresh fallback's `PromptLine` goes unsent after a `Wait` dies: fixed (the bound sentence names `DeliverPendingPrompt`, which types an unsent `PromptLine` past the recorded offset, `internal/shuttleengine/run.go:900`).
- fix-proposal, MEDIUM, activity and session-state readers report the stale limit during the relaunch: fixed (they report a relaunching run with no turn-end facts; a criterion was added).
- fix-proposal, LOW, a pane that never reaches ready can end as a timeout: fixed (step 4 and the criterion name both outcomes).
- fix-proposal, LOW, step stop past the bound: fixed, merged with fix-claims' second finding (`StopStepRuns` stops a relaunching run within its bound or past it).
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
