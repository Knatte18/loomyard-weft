# Card 12 — drift-activity

**Edit:**
- `internal/loomcli#agentActivityFor`
- `internal/loomcli#TestAgentActivity_LiveLockWaitReadsAsActivity`

**Uses:**
- `internal/driftengine#Active`
- `internal/driftengine#NewPaths`

**Intent:**
Make a held capture lock or compare marker read as activity now to the loop's step idle watchdog (D4), so `step_idle_timeout_min` never kills `Drift-Await`'s wait, darn's foreground capture or a mechanical compare.

`agentActivityFor` checks `driftengine.Active(driftengine.NewPaths(loc.AnchorPath()))` right after the parent-lock wait and before the shuttle reads, returning `time.Now(), true` when it is true and its error when the probe fails, as it does for a live verify marker and a live parent-lock wait.
A lock a dead process left reads free, so a dead capture or compare reads as none.
What this lets through, and its bound: while a capture holds its lock, another row's hung agent is not killed as idle either, until the capture ends; the capture's own timeout counts from its command's start, so a capture queued for a gate slot keeps the watchdog off until it gets one and for at most `capture.timeout_min` after (D4).
Its doc comment names the two new readings.

`TestAgentActivity_LiveLockWaitReadsAsActivity` gains rows: a capture lock held by the test reads as activity; a compare marker held by the test reads as activity; a capture lock file with no holder reads as none; a capture lock path the test occupies with a directory, which the probe cannot take, returns the probe's error.

**ImpactSummary:** One more reading in the activity closure every loom and darn loop uses; no other caller.
