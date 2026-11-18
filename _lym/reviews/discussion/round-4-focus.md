---
round: 4
focus:
  - "Decision D7 step 4 in `_lym/discussion/decision-record.md`: settle whether the relaunched pane's startup wait can decide on the pane capture alone, given the run's events file (`RunState.EventsPath`) is kept and already holds an earlier session-start and turn-start line, against `awaitStartup`'s `checkLivenessTick(..., eventsProof=true)` short-circuit in `internal/shuttleengine/wait.go`. Does a trust or login dialog on the new account get dismissed, and does a pane that never reaches ready end the run as a strand that died?"
  - "Decision D7's order of writes in `_lym/discussion/decision-record.md`: settle what state a record is in if the Wait process dies after reed relaunches the pane but before the relaunch writes the session id, `PromptLine`, prompt offset and copy fact and clears the relaunching mark. Does the bound that ends a relaunching record cover a live strand, and does an attach then read the new launch's facts rather than replaying the limited turn end?"
  - "Decision D7 and its PATTERN bullet in `_lym/discussion/decision-record.md`: settle which readers of a run record treat a relaunching record as running. Check every site that compares a record's outcome to `runOutcomeRunning` in `internal/shuttleengine` (the shed stop chain's `StopStepRuns` in `stepstop.go`, `dispositionCandidate` in `attach.go`, `activity.go`, `sessionread.go`, `unfinished` in `carryover.go`), and whether a step stop during the window leaves the run stopped with no strand up."
---

Each entry names a D7 relaunch-window question still unsettled after round 3.
Do not flag the optional "Notes for the plan writer" subsection, missing rejected alternatives in `decision-record.md`, or incomplete call-site enumeration.
