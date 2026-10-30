# Dobby sends a running child's run notices, not batten's Run-Shed row

The quiet, driver-dead, api-error and usage-limit notices of a running child now come from the dobby that drives the run, not from batten's Run-Shed row.
Batten is a profile of a shed that dobby drives, and these notices need a continuous watch over a run whose state is not changing, which is the daemon's job.

## What changes

- `internal/dobbyengine/runnotice.go` holds the detection, moved from `internal/battenshed/notice.go` with its checks, their order, the episode keys, the quiet window, the two-minute API-error idle, the idle clocks that leave out recorded sleeps and paused intervals, and the api-error revive streak.
  Each probe tick runs it, after the scheduling steps, for every listed run whose batten state is running with no cancel marker.
  A child that is not running, or that a pause covers, is in no condition.
- dobbyengine declares its own reading types (`ChildReading`, `AgentRun`) and seams (`RunNoticeDeps` on `Deps.RunNotices`), since its import allowlist admits neither battenshed nor shedengine.
  `internal/battencli/dobbynotice.go` fills them from the reads batten's wiring already made: the child status, driver liveness with the driver and run locks, the agent activity with its wait markers, the pause coverage, `tendercli.ReviveStalled`, and whether batten respawned the child the cap of times (`battenshed.RespawnCapReached`).
  `internal/beamcli/dobbydeps.go` wires them, reading the quiet window from the Run-Shed row's `notice_quiet_min` as it already reads `notice_probe_s`.
- A moved notice reads `[dobby notice] <slug>: running at <row>; <condition>; …` and goes through dobby's existing Notice seam, from `<shortname>:DOBBY` or `<shortname>:<beam>:DOBBY`.
- The marker of the notices sent moved from batten's scratch directory to the run's records directory under `_lym` (`shedrun.NoticeEpisodeFile`), and batten's status commit carries it beside the status and seed, so it reaches the next machine.
  The child's state, its since and the revive streak ride in the run's entry of dobby's state record, so a re-exec sends nothing twice.
- Batten keeps only the notices that follow a step its pass takes: the state change with its report, arbiter and relay holds, the driver-not-revived follow-up of a halted child, the busy-pane notice and the owner notice, all still `[batten notice]`.
  `InnerRunDeps` loses `Activity`, `Sleeps`, `ReviveAPIError` and `NoticeQuiet`, and battenshed loses `AgentActivity`.
- The orch role stencil lists the `[dobby notice]` lines and counts them as stops.
  The battenshed, dobbyengine, orchcli and shuttleengine docs and `docs/overview.md` follow the move.

## Design decisions

- The marker is emptied rather than removed when its keys end, so a tracked marker is never left as a deleted file that batten's status commit, which stages it only when present, would not commit.
- Delivery attempts are kept in process: a notice not posted is tried on each probe tick, at most three times, then dropped with a Warn.
  A missing orch strand counts as a failed attempt, since dobby's Notice seam reports it as an error.
- A child lock that cannot be read makes the driver read as alive, logged at Debug, as batten treated it.
- `innerRunEntry` still validates `notice_quiet_min` on the Run-Shed row, though no producer reads it now: the row stays the one place the key is set.
- The batten tests of the moved conditions are replaced by dobby tests over a fake child reading; batten's tests now assert that a running child gets no notice from the row and that the respawn cap is readable from the memory record.

## Bounds

- Between a marker write and the next batten transition, the marker is an uncommitted file in the prime, as the status file already is while a run is watched.
- A machine that takes a run over dates the child's since from its own first sight, so a driver-dead notice the other machine sent can go once more.
- No run notice goes while an anchor change stands on the dobby's records worktree or the prime's.
