## Changes

- F1 — fixed differently: `newestTurnEnd` (`internal/shuttleengine/activity.go`) now calls `unreadableFiles.Recover` only on a success return (a turn end found) or after both parts parsed with none found, never between the two parses.
  The proposed change (parse both parts first, then recover once) was not made: it would parse the pre-offset part on every poll even when the own part already holds a turn end, while the chosen change gives the same single-episode result without that extra parse.
  Added a subtest to `TestReadAgentActivity` (`failingPartEngine` fails only the pre-offset part): across four reads it asserts one Warn and no recovery Info.
- F2 — fixed as proposed: added a row to `TestReadAgentActivity` with events `STOP:skill\n` and a prompt offset past the end, asserting the transcript's last activity is reported and no API error.

Commits: `ab451e53a` (F1, F2), and a follow-up correcting the F1 comment after the self-check.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW: the recovery comment in `newestTurnEnd` described the proposed change, not the code — fixed as proposed (comment reworded).
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
