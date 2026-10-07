# Card 2 — waiting-trace

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/waiting_test.go`

**Intent:**
D3: a waiting turn end names what it waits on in the durable trace.

When `pollEventsTick` records a waiting turn end (`recordWaiting`), it logs once at Info through `internal/logger`, naming the strand guid and, for each outstanding task, its kind, id, label and `Signal`.
It is once per turn end: a tick with no new bytes, and `expiredTurnEnd`'s re-checks, log nothing.
No bound changes: a non-awaited shell still expires after `background_shell_wait_min`, and a fork never expires.

Test in `waiting_test.go`, as an assertion in `TestPollEventsTick_Waiting` through `internal/testkit/logcapture`: one waiting turn end followed by idle ticks logs exactly one line carrying each task's kind, id, label and signal.

**ImpactSummary:** Adds one Info trace line per waiting turn end; no classification changes.
