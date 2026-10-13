# Fixer report — discussion, round 3

## Changes

- F1, fixed as proposed: D5's conditioned send waits, after its landing, for a `turn_start` past the pre-send events offset, at most one submit window, and revive holds the pause record lock across that wait.
  The revive-versus-pause sentence now states the guarantee from the turn start on, and names the two unrecorded cases: a send landing with no turn start seen, and a revived turn ending before the pause's poll, both left to a later `revive`.
  The report gains "sent with no turn start seen".
  Scope In's conditioned-send bullet and the `shuttleengine` doc constraint name the wait, and the Acceptance criterion on a pause during an in-flight line matches the rewritten bound.
- F2, fixed as proposed: decided that the conditioned send refuses at once on every hold its ready reading names (draft, operator activity, unmatched turn start).
  The Bound states the longest lock hold per send, two submit windows, and that `pause`, `resume` and the orch watcher's record write wait at most that long.
  An Acceptance criterion pins the at-once refusal; the support log records the pick as Q13.
- F3, fixed as proposed: D6 and its cancellation criterion qualify the during-the-hold cancellation with "before its own deadline passes", and D6 states that a later cancellation is not observable once the deadline has passed during the hold.
- F4, fixed as proposed, first option: D8 re-execs only at a tick with no live hub pause entry at all, so no handled entry passes to the new image; the re-exec criterion and the support log (Q14) match.

## Disputed

None.

## Deferred

None.
