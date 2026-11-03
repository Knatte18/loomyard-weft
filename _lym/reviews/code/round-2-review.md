---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/wait.go:933
    summary: the shell nudge is queued before expiredTurnEnd decides, so a waiting turn end whose shells are already past the bound counts Done and still leaves a "run does not count as done" line queued
    fix: in pollEventsTick, call run.expiredTurnEnd() first and call run.queueShellNudge() only when it returns an empty outcome, for example `outcome, held, err := run.expiredTurnEnd(); if outcome == "" { run.queueShellNudge() }; return outcome, held, err`. Add a row to TestWait_GatedShellNudge, or a pollEventsTick case, where the shell's shellFirstSeen is already a bound old and the first waiting turn end counts Done with no SendText call.
  - id: F2
    severity: NIT
    class: consistency
    location: internal/websterengine/doc.go:641
    summary: the doc says "With both present", pinning the count of Master's output files, which the prose rule forbids because the count changes independently of this paragraph
    fix: in internal/websterengine/doc.go line 641 replace "With both present" with "With every output file present".
---

### [LOW] Shell nudge is queued even when the same tick counts Done

**Location:** `internal/shuttleengine/wait.go:933`

**Issue:** In `pollEventsTick`, the gated shell-only branch with every output file present calls `run.queueShellNudge()` and then `run.expiredTurnEnd()`.
`shellFirstSeen` survives a turn start, so a shell first seen in an earlier waiting turn end can already be past `background_shell_wait_min` when a later waiting turn end lists it.
That tick then returns `OutcomeDone` from `expiredTurnEnd`, and the nudge is queued anyway.
When the gate then fails with budget left, `handleGatedBoundary` returns unfinished after sending the re-prompt.
`sendShellNudge` runs right after and types the line "The run does not count as done while one runs" into the pane, on top of the re-prompt, though the run has just counted the turn end Done.
It needs an earlier nudge that was dropped under a pause or failed, plus a later waiting turn end over the same shell, so it is rare.
The wording in the pane is wrong when it happens, and the extra send can fail busy against the re-prompt.

### [NIT] Doc pins the count of Master's output files

**Location:** `internal/websterengine/doc.go:641`

**Issue:** The changed paragraph says "With both present", which fixes the number of Master's output files in prose.
The count changes independently of this paragraph, so the sentence goes stale silently when an output file is added or removed.

## Rejected

- Focus 1, clearing the wait on a later turn start: the clear at `wait.go:903` runs before any expiry in the same tick, and a turn start in the same batch as the waiting turn end is handled at `wait.go:927`. No Done is counted mid-turn for an engine that reads turn starts. `claudeengine` is the only production implementation of `ParseSessionSignals`. An engine without turn-start parsing keeps the old at-the-bound Done, which no production engine has. `TestPollEventsTick_TurnStartEndsTheGatedShellWait` fails without the clear.
- Focus 2, nudge send order: `nudgedShells` is written only after `sendWithin` succeeds, and `sendShellNudge` clears the queued line first, so a dropped or failed send is neither retried in place nor marked named. This is not a defect.
- Focus 3, test rows: `TestWait_GatedShellNudge` reaches the `turnStartAfter` guard (the row seeded with a turn start), the pause drop (resume, then queue again, one send), and the failed send (two sends with an idle tick between, so a retry in place would make three). The Done-at-the-bound path is covered by the gated row in `TestWait_RecordsEndedShells`. No behavior in the diff is left without a failing test if removed.
- Focus 4, doc comments: `shellWaitActive`, `AwaitedShellPrefixes` and `masterAwaitedShellPrefixes` match the code. The gated shell-only waiting turn end is held up to `background_shell_wait_min` with every output file present, and judged as a plain turn end only with an output file missing.
- The tests ran through `lym gate test ./internal/shuttleengine`, which reported the tier carried with no failures.
