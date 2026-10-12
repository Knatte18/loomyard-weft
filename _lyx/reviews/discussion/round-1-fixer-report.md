# Round 1 fixer report — discussion

Each finding's premise was checked against the code: `internal/shedengine/status.go` declares `running`, `paused`, `done`, `blocked`, `failed` and `awaiting`; `internal/shuttleengine/agentwait.go` counts `asking` or `idle-stalled` or a live held marker as waiting; `internal/reedengine/namerepair.go` calls `pasteSubmitLocked` directly; `internal/seatengine/work.go` checks the up line before the session reading.

## Changes

- F1 (fixed as proposed): D1 and the second acceptance criterion name `blocked`, `awaiting` and `failed` in place of `halted`.
- F2 (fixed, the "yes" branch): D2 adds `asking` to the held readings in the Decision and the third acceptance criterion, states that the session reading alone decides and the held-turn wait marker is not read, and the Rationale says the held set matches shuttle's waiting reading.
- F3 (fixed as proposed): D3's no-cap sentence is replaced by the stdin bound, naming shuttle's pre-write guard and reed's activity check per caller.
- F4 (fixed as proposed, recommended rule): D4 forwards every `-v` or `--verbose` word before the first `--` to each argv, the rest to the last argv; the trailing-words criterion adds `lyx -v prime` running both commands with `-v`.
- F5 (fixed as proposed): the pre-run item moved from Open risks to Constraints with the corrected count (once for the shortcut command and once per argv); no Open risks line kept, since no hook was found non-idempotent.
- F6 (fixed as proposed): D4's board-wording departure sentence deleted.
- F7 (fixed as proposed): the stray "died" removed from the D2 acceptance criterion.
- F8 (fixed as proposed): Q2 in the support log names running, batten-paused or starting covered runs.

## Disputed

None.

## Deferred

None.
