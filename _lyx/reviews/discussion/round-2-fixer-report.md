# Fixer report — discussion, round 2

## Changes

- F1, fixed: D4 states that a covered waiter yields its turn, keeping its ticket and place while a free record goes to the oldest live ticket whose run no entry covers, with each ticket naming its run's slug, and that a covered waiter still first at the resume is granted next.
  D3 step 4 and D5's parent-lock bullet name each covered waiter and the runs queued behind it.
  A Holds-and-readers criterion pins an uncovered run behind a covered waiter being granted under a per-run entry.
- F2, fixed: D10 picks the new-field option: `run.json` gains a field naming the shed run directory and step that started it, written at shuttle's Start and read by the arming module.
  The bound says why the driver's strand and the orch fall outside it (no step starts them, so their records carry none), and a live run with no step field is left and named in the halt note.
  The PID option was not taken: an attach rewrites `RunState.PID` to the attaching process, so it does not identify the starting step.
  The last Sleep-and-#567 criterion names the tie.
- F3, fixed: D7 ends the orch's grace by D3 step 2's rule, early only once the orch reads idle and has no network operation marker open, and otherwise at the recorded grace end.
  The orch criterion now says a busy orch running a local command is interrupted only at the grace end.
- F4, fixed: the refusal's way forward is asking its parent session, in D1, the Caller criterion and the Constraints refusal entry.
- F5, fixed: D1 states that caller identity is read from the strand-name environment variable, that the agents' settings deny commands naming it, and that the check guards against a mistaken call, not a determined agent.
- F6, fixed: D4 states that a call held at entry prints one line, to stderr or its log, naming the pause and `lyx resume` once when its wait starts; a Holds-and-readers criterion pins it.
- F7, fixed: the first Boundary-pause criterion and D8's landingshed bullet state that `--after` naming a row waiting for the parent lock ends that wait with the halt of D8.
- F8, fixed: support-log turn 10's who-may-call line says D1 bounds an agent caller to its own pair's slug and refuses the rest.
- F9, fixed: the `_No rounds yet._` seed line is deleted.

## Disputed

None.

## Deferred

None.
