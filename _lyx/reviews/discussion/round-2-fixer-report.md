# Fixer report — discussion round 2

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise confirmed: `removeStrandIfLive` (`internal/shuttleengine/stop.go`), `StopStepRuns` (`stepstop.go`) and `removeSupersededStrands` (`attach.go`) skip a strand that is not live, and `Wait` removes a strand only on `OutcomeDone` without `KeepPane`.
  D5 now decides that shuttle's stop path (stop verb, `StopStrand`, step stop, supersession, and so a seat table's stop) removes a tracked own-window strand whose pane is present but dead, bounded to the strand a run record names, with window 0 strands keeping live-only removal.
  For the prime and runs with no later stop, `Wait` finalizing `OutcomeDied` on an own-window strand saves the pane's last capture into the run directory and removes the strand; a timed-out run keeps its strand, since its agent may be live and a later attach may take it up, and its window then stays until a stop, a respawn, `lyx reed remove` or a pair teardown, which is recorded as accepted.
  Acceptance lines added for a step stop and a respawn of a dead agent's run, for `Wait`'s died removal, and for a seat table whose advisor died.
  The plan-writer notes name the three shuttle sites.
- F2 (MEDIUM), fixed as proposed.
  D3 takes remove out of the failing ops: it kills the pane, ends the window, skips only the even layout and reports the invalid key.
  The acceptance line now lists up, add and resume, with a new line for remove under an invalid key.
- F3 (LOW), fixed as proposed.
  D2 counts strands bound to a present pane, alive or dead, toward the live count, and drops the group's recorded choice and `WindowSize` with its last strand; an acceptance line covers both.
- F4 (LOW), fixed differently.
  The proposed rule, Resume relaunching an opened strand only when its pane was alive when the server died, needs a record reed does not keep: after a server restart every pane is gone, and `planResumeLaunches` (`internal/reedengine/lifecycle.go`) only sees liveness and done-when paths.
  D12 instead chains an exit-marker write before the self-removal and makes the marker the strand's done-when path, so a failed self-removal leaves a dead pane bound until `lyx reed remove` or a pair teardown, and `Resume` drops the strand through the existing done-when drop.
  The Bound and the open verb's acceptance line state the failed-removal case.
- Support log: round 2 recorded under Review rounds, and Q12 updated to D5's removers.

## Disputed

None.

## Deferred

None.
