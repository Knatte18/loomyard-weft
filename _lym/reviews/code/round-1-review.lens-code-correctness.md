```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. Nothing the diff changes is wrong, so I have no findings. The list is per function and branch.

**`readTaskLimitAt`**
- Cgroup `/a/b`: levels run `/a/b`, `/a`, `/`, then it stops on `/`. Correct.
- Cgroup `/` (root): `/` is read once, has no `pids.max`, and the result is not found. Correct.
- Cgroup `/../x` or `/a/../..`: `path.Clean` collapses these to `/x` and `/`, so the walk cannot leave the mount. Correct.
- Tie between two levels: the strict `<` keeps the deeper one. Correct.

**`readCgroupV2Path`**
- Missing file: not found. Correct.
- v1-only lines such as `12:pids:/x`: no `0::` prefix, so not found. Correct.
- `0::` followed by a non-absolute path: rejected, not found. Correct.
- Hybrid file with v1 lines before the `0::` line: the `0::` line is found. Correct.
- Line longer than the scanner's 64KB buffer: `Scan` stops and the result is not found. A cgroup path that long cannot occur, so this is harmless.

**`readPidsLevel` and `readIntFile`**
- `max`: `Atoi` fails, so the level is skipped. Correct.
- Malformed `pids.max`: skipped. Correct.
- `pids.max` readable but `pids.current` missing or malformed: skipped. Correct.
- Trailing newline: `TrimSpace` handles it. Correct.
- A negative or `current > max` reading cannot come from the kernel. Not a concern.

**`TaskLimit.Headroom`**: `Max - Current`, correct.

**`taskLimitHolds`**
- `headroom == 0` or a nil seam: nothing is read, `held` is false. If an episode was open, `starts resume: no task limit applies` is logged once and the flag clears. Correct.
- Found and `Headroom() < headroom`: logs the hold once on entry and stays silent while it continues. Correct.
- Headroom exactly equal to `task_headroom`: not held, because the comparison is strict. This matches "fewer than".
- Found, with headroom back above the margin and an episode open: logs the resume with both numbers, then clears the flag. Correct.
- Not found with an episode open (the reading failed during a hold): logs `no task limit applies` and releases the hold. This is consistent with D2, "no readable limit reads as no hold".
- The flag is process-local and the state record is untouched. Correct against D5 and `PATTERN-tolerant-records`.

**`startReady` hold branch**
- Placement is after the `max_runs`, flight, pause, ready, fingerprint-reset and `NextTry` checks, and immediately before `running++` and `startFirstPass`. This matches D4.
- A held task gets no `updateStart` call, so no record, notice or backoff is written. Correct.
- `break` ends the tick's starts, as D4 asks. `d.busy` is still set from `len(d.flights)`, so the existing flights are unaffected.
- The fingerprint-reset `updateStart` can run before the hold. That clears an old refusal record and is harmless.
- When no task reaches the check, `taskLimitHeld` is not updated. An open episode then stays open until the next start attempt, so the resume line comes late. This matches "the first start the reading lets through".
- Retry after a hold: the idle timer doubles up to `idleCeiling`, one minute (`loop.go:26`, `loop.go:628-633`). The hold writes no state, so the next tick comes within a minute. The doc's "within the idle timer's minute" claim holds.

**`LoadConfig`, `ConfigSpec` and the template**: `task_headroom` is registered, covered by the negative check, and defaults to 500. `0` passes. A negative value is refused naming the key. Correct.

**`hubDobbyDeps`**: `proc.ReadTaskLimit` has the signature `func() (TaskLimit, bool)`, which matches `Deps.TaskLimit`. Correct.

Tool calls:
1. `grep` of `internal/dobbyengine/loop.go` for the idle timer and busy handling. My context lacked how a tick is rescheduled after a held start, and it confirms a retry within the one-minute ceiling.
