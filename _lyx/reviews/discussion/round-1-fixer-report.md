# Fixer report — discussion, round 1

All eleven findings were checked against the code and found valid; all are fixed.

## Fixed

- **F1 (BLOCKING), D5.**
  The `failed` write now goes through one `internal/shedengine` function.
  That function writes only while it holds the run lock the busy refusal reads, taken without waiting, and only while the file still reads `running`.
  When another holder has the lock, the function writes nothing and the loop stops as `busy`, so `State` and `Error` stay Shed-written (`internal/shedengine/status.go`'s ownership doc).
  Added an acceptance row for a failure stop under another holder's lock, and a Constraints line for the `Status` ownership contract change.
- **F2, D3.**
  `--clear` now removes only the two conditions and leaves `pause_requested` alone, with a stated bound, so Shed stays the only writer of `false`.
  The condition fields are declared shared write-to-clear, cleared by Shed only in the `paused` persist and in a `goto`'s persist.
  The acceptance row now checks that a recorded bare request survives `--clear`.
  The support log's ledger entry records the narrowing.
- **F3, D2.**
  Liveness is now a loop lock (`internal/lock`) that the loop holds for its whole life, so a stale record or a reused pid never reads as live.
  The spawn is exclusive: a loop that cannot take the lock exits before any step.
  The pid file records the pid and the start time (`internal/proc`'s `StartTime`) and is used only for kills, which act only while the lock is held and the start time matches.
  Both cases were added to the acceptance row on a second `--until-stop`, and the loop lock was added to the Constraints file list.
- **F4, D2.**
  An unwatched envelope is now delivered only while its `loop.current_producer` and `loop.history_length` match the status file.
  A stale envelope is retired undelivered and a fresh loop is spawned.
  Added to the acceptance row.
- **F5, D1, D4, D7, D8, Acceptance criteria.**
  D1 now names every key in the closed `loop` set: `stop`, `first_producer`, `status_moved`, `restep`, and `trace_copy`/`stderr_path` at top level.
  It also maps a bare pause to `halted` and a fired condition to `stop-condition`, and states how the loop fills the short error form for an `interrupted` stop.
  D4, D7 and D8 now point at those keys instead of carrying their own lists.
  While fixing this, I found that `internal/logger` exports `LYX_TRACE_ID` to every spawned child.
  Without an explicit decision, every child would inherit the loop's trace id, which breaks D1's claim that each child keeps its own per-step trace, and the interrupted error form needs the child's trace file.
  D1 now has the loop mint a fresh trace id per child and hand it through `LYX_TRACE_ID`, with an acceptance row.
- **F6, D11.**
  The reviewer's fix, killing the loop before the quiet wait, would defeat `fabric remove`'s policy of never ending a busy driver (`internal/pairteardown/doc.go`, Caller policies).
  The fix applied instead closes the same window:
  - The quiet rule counts a held loop lock as busy.
  - The session-end step kills the loop as its first act, before the reed session ends.
  - The stated bound is that no step starts once the session-end step begins.

  The Scope bullet and the acceptance row were updated to match.
- **F7, D5.**
  `PATTERN-failure-stop-visible` is now scoped to runs driven by `--until-stop` with no other run-lock holder, and the Go driver and a bare `step` are named as outside it.
- **F8, D3.**
  A `goto` now clears both conditions, and a condition otherwise lasts until it fires or is cleared.
  The goto case was added to the acceptance row.
- **F9, D8.**
  The count is now consecutive: a failing re-step stops the loop and a success resets the count, with a stated bound.
  The acceptance row was updated.
- **F10, Added after Discussion.**
  Moved into `## Constraints` as one line that keeps the 16200 ceiling and the 16384 cap and drops the current byte count.
  The run history moved to the support log's Question ledger.
- **F11, D10.**
  Dropped D10's restatement of the stderr routing; D4 keeps it.

## Disputed

None.

## Deferred

None.
