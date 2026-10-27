# Card 9 — dobby-change-log-heartbeat

**Edit:**
- `internal/dobbyengine#Run`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#daemon.loop`
- `internal/dobbyengine#daemon.beamOwnedElsewhere`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#daemon.spawn`
- `internal/dobbyengine#daemon.passExited`
- `internal/dobbyengine#daemon.spawnTidy`
- `internal/dobbyengine#daemon.settleTidy`
- `internal/dobbyengine#TestRun_TickFailuresLogOncePerEpisode`
- `internal/dobbyengine#TestCountLandingsAndTidy`
- `internal/dobbyengine#tidyHarness.daemonOver`
- `internal/dobbyengine/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/dobbyengine#TestLog_OneLinePerStartAndPass`
- `internal/dobbyengine#passKind.String`

**Create:**
- `internal/dobbyengine#heartbeatInterval`
- `internal/dobbyengine#daemon.heartbeat`
- `internal/dobbyengine#TestLog_ChangesOnly`
- `internal/dobbyengine#TestRun_Heartbeat`

**Uses:**
- `internal/dobbyengine#newPassHarness`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#newTidyHarness`
- `internal/dobbyengine#runningRun`
- `internal/dobbyengine#readyTask`

**Intent:**
Decision 6: `dobby.log` records changes only, plus a ten-minute heartbeat, every line through `logLine` (card 6), riding the loop's existing ticks with no new wait or wake (PATTERN-daemon-wakeups).

Lines, each one `logLine` call:
- `spawn` no longer logs `pass <slug> <kind>`, and `passKind.String`, whose doc names that line and whose one caller it was, is deleted; `startFirstPass` keeps `start <slug>`.
- `drivePasses` compares each listed run's batten state and producer, read before the done filter so a run turning `done` is seen, with `daemon`'s new in-memory map of the states it last logged: a change logs `run <slug>: <old state> -> <new state>`, with ` at <producer>` for a state other than `running` and `done`; a run first seen logs `run <slug>: <state>` once; a run no longer listed leaves the map. The map lives in memory only, so a re-exec logs no change it did not see.
- `passExited` logs `pass <slug> failing: <error text>` when the failure starts an episode, that is when the count it records is 1, and `pass <slug> succeeds again` on an exit 0 whose record held a failing count, reusing the count the notice step keeps.
- `loop` logs `tick failing: <error>` on the first failing tick of an episode and `ticks succeed again` on the tick that ends it, and `beamOwnedElsewhere` logs `owner unreadable: <error>` on the first unreadable read and `owner reads again` on the recovering one, through two `daemon` booleans beside the existing `failures` and `ownerRead` episodes, which keep their `logger` lines.
- `spawnTidy` logs `board tidy started over <n> landings: <report path>`; `settleTidy` logs `board tidy report <path>` when it posts the report and `board tidy timed out: <slugs>` on the timeout.

Heartbeat: `heartbeat` writes `heartbeat: <slug> <state> <producer>; …`, one item per run listed, or `heartbeat: no runs`, and records the time. `heartbeat` reads `Deps.Runs` itself, dropping done runs as `drivePasses` does, and a failed read writes `heartbeat: runs unreadable: <error>`. `Run` writes one right after the start line, which also seeds the logged-state map; `loop` writes one after the first tick at least `heartbeatInterval` after the last, whatever that tick did, so a tick that a standing anchor move keeps from driving passes, or whose runs read failed, still writes it. A dead dobby so shows in its log within the interval plus the longest gap between ticks.

Tests (tier0, injected clocks):
- `TestLog_ChangesOnly` replaces `TestLog_OneLinePerStartAndPass` over `newPassHarness`: a tick that spawns passes and starts task `a` logs `start a` and no `pass` line; a run whose state the fake turns from `running` to `blocked` at `Plan` logs one change line naming the producer, and an unchanged next tick logs none; three failing exits with one text log one `failing` line and an exit 0 after them one `succeeds again` line; a first-seen run logs its state once.
- `TestRun_Heartbeat` over `newLoopHarness` with one listed run: the start writes a heartbeat naming the run's state and producer; with a re-exec record and that run unchanged, the first tick logs no change line; a tick nine minutes later writes no heartbeat and one at ten minutes writes the second; with no runs the heartbeat says `no runs`.
- `TestRun_Heartbeat` also has two rows at the ten-minute tick: one with an anchor-move record standing from the start still writes the heartbeat, and one whose `Runs` fails from that tick on writes `heartbeat: runs unreadable` naming the error.
- `TestRun_TickFailuresLogOncePerEpisode` also asserts one `tick failing` and one `ticks succeed again` line through `Deps.Log`, and an owner seam failing twice then succeeding logs one `owner unreadable` and one `owner reads again` line.
- `TestCountLandingsAndTidy` asserts the spawn, report and timeout lines, through a recording `Log` that `tidyHarness.daemonOver` fills.

`internal/dobbyengine/doc.go`'s loop bullet on log lines is rewritten for these lines and the heartbeat; `docs/overview.md`'s beam entry, which says dobby writes a line per task start and per pass, now says it logs changes and a ten-minute heartbeat.

**ImpactSummary:** Log lines change throughout the loop and the tidy step; no scheduling decision reads them, so only the log tests change.

**Commit:** 9: Log only dobby's changes and a ten-minute heartbeat
