```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, with one tool call to check `WriteFailedStop`.

**Per function and branch: case run, correct or not**

- `gitexec.runSpawning`
  - Start fails with EAGAIN twice, then succeeds: 3 starts, delays 250 ms and 500 ms, returns the success. Correct.
  - EAGAIN on every try: 4 starts, 3 waits (250, 500, 1000 ms), returns the last EAGAIN error. Correct.
  - A git that exits non-zero gives `*ExitError`/`GitError`, which is not EAGAIN, so it runs once. Correct.
  - Stall retry: `runSpawning` sits inside the existing stall retry, so the retries multiply but each is bounded. Correct.
  - Context cancel is never EAGAIN, so it is never retried. Correct.
- `proc.NoteSpawnAgain`, `IsSpawnAgain` and `LastSpawnAgain`
  - nil error returns false and stores nothing. Correct.
  - A zero store reads back as `(zero, false)`. Correct.
  - The record is a single atomic int64, so it is safe under concurrent callers. Correct.
- `reedengine.retryExecAgain`
  - nil error: `NoteSpawnAgain` is false, so it returns at once. Correct.
  - EAGAIN on the last try: it returns the error after noting it. Correct.
  - Other errors return unretried. Same behavior as before apart from the shared predicate. Correct.
- `shedtransient.Class`
  - The `ErrNotStarted` row still comes first, so existing rows are unchanged.
  - A bare EAGAIN error gives `host-resources`.
  - `ErrAgentDied` with no noted refusal gives `""`.
  - `ErrAgentDied` with a refusal under 2 minutes ago gives `host-resources`.
  - A refusal exactly at the window edge counts, because the test is `<=`.
  - Correct.
- `multillm.mapOutcome`
  - `ErrAgentDied` is wrapped only for `OutcomeDied` that got past the not-started branch above it. Timeout is unchanged. Correct.
- `shedverbs.runLoop`, interrupted branch
  - The failed write now comes before the hook, and the hook still runs when the write returns Busy. Correct.
  - A status file the child moved on gives a new `lastKnown`. Correct.
  - Finding 1 below: the status file is found but its `CurrentProducer` is empty.
- `shedverbs.stderrExcerpt`
  - Empty file: `lines` is nil, the slice is `lines[0:]`, result `""`. Correct.
  - Crash line present: three lines from the first crash line, bounded by `min`. Correct.
  - Unreadable file gives `""`. Correct.
  - A panic's second line is the `goroutine N [running]:` stack header, so the excerpt often holds the header and one frame. That is within "first lines".
- `shedverbs.interruptedTraceFile`
  - A trace file exists: the first one is used.
  - No trace and a non-empty stderr file: the stderr path.
  - No trace and empty or missing stderr: `""`.
  - `StopFiles` nil: `""`.
  - Correct.
- `loopObject.PauseReason`
  - Both `runLoop` and `loopExitStop` fill it, and it is emitted for every state.
  - The driver reads it only when the loop's `State` is paused and its own reason is empty, so an absent key decodes to `""`. Correct.
- `loomAfterInterrupt`
  - Status unreadable: `failed` with the plain cause. Correct.
  - Status holds an error that already contains the cause (the stop's own write): no extra line. Correct.
  - Status holds the child's own error: it is appended on a `status error:` line. Correct.
- `ClassifyHalt`
  - `failed` or `paused` plus `interrupted` gives `interrupted-stop`. Correct for those two states.
  - Finding 2 below covers the other states.

**Findings**

1. LOW, `design`: the blamed producer can go blank.
   - Location: `internal/shedverbs/loop.go:296-298`.
   - `lastKnown` is overwritten with `after.CurrentProducer` whenever the status file is found. The decision record says to fall back to the producer read before the child when the after-read gives none.
   - A found status with an empty `CurrentProducer` gives an empty blamed row in the cause, the hook argument and the note.
   - Evidence: `loop.go:296-298`; the record's "falling back to the one read before it".

2. LOW, `design`: `ClassifyHalt` covers fewer states than the record states.
   - Location: `internal/loomengine/anomaly.go:90`.
   - The record says `interrupted-stop` applies "whatever non-blocked state the status holds". The code accepts only `failed` or `paused`.
   - `WriteFailedStop` (`internal/shedengine/failedstop.go:72-74`) leaves any non-`running` state untouched, and returns Busy or an error without writing.
   - A child that finished the run (`done`) and then died, or a failed write that leaves `running`, gives an interrupted halt note with no anomaly.
   - Evidence: `anomaly.go:90`; `failedstop.go:55,72-74`; `looparm.go` note state comes from the status file.

**Tool calls**
1. `grep` for `WriteFailedStop` in `internal/shedengine/failedstop.go`: my context lacked it, and it decides what state the hook sees for a paused, done or busy run.
