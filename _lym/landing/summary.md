# reed: wait for an entered session's goroutines in test cleanup

`TestEnterSession_DrivesTheToldTmux` in `internal/reedcli` failed once in a tier1 gate run,
in `TempDir` removal with `directory not empty`, and passed 30 of 30 repeats afterwards.
The cause, found by reading the code, is that the test's cleanup canceled the session's context and returned at once.
The OOM watch goroutine could then still be inside its first strand read, which writes under the worktree.

Closes #266

## What changes

- `watchedSession` carries a `running` `*sync.WaitGroup`.
  `enterSession` starts the OOM watch and, when there is one, the name-repair loop through it.
- A new test helper, `stopWatchedSession`, cancels a session and waits for those goroutines to return.
  It reports a test error if they have not returned within 30 seconds.
- Both tests that call `enterSession` clean up through the helper.
- `TestEnterSession_DrivesTheToldTmux` also asserts that `.lym/reed.lock`, which the OOM watch's first read creates,
  is absent before `enterSession` and present once the wait returns.

## Design notes

Only tests wait on `running`.
The daemon still calls `cancel` without waiting, so its behavior is unchanged.

## To check

- The cleanup failure was never reproduced, so nothing shows by observation that this change ends it.
  The cause rests on reading the code.

## Merge danger

**Door:** two-way
**Blast radius:** reed
