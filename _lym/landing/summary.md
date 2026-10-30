# Dobby sends the running-run notices as [dobby notice] lines

Dobby now detects and sends the notices for a running child itself, in its probe-tick loop over the runs it drives.
Batten's Run-Shed row no longer detects them.
Batten is a profile of a shed and drives nothing; dobby is the common driver of every run under its beam and already reads each run's state, so the watch belongs there.

## What changes

- Dobby sends four notices for a running child: a dead driver, an API-error stall, a usage limit and a quiet child.
  They read `[dobby notice] <slug>: <condition>; …` and go through dobby's Notice seam, from `<shortname>:DOBBY` or `<shortname>:<beam>:DOBBY`, to the nearest live orch.
- The api-error revive that precedes an API-error notice moves to dobby with it.
- Batten keeps the notices tied to a step its own pass takes or holds, under `[batten notice]`: state-changed with its holds, driver-not-revived, busy-pane, owner-elsewhere, and the Land, Worktree-Teardown and hand-back stops.
- The orch role stencil (`contracts/stencils/orch/orch-template-role.md`) lists the `[dobby notice]` lines and counts them as a stop.
  `docs/overview.md`, the `battenshed`, `dobbyengine`, `orchcli` and `shuttleengine` package docs and the `notice_quiet_min` recipe comments follow the move.

## Design decisions

- Detection lives in a new `internal/dobbyengine/runnotice.go` and runs on each probe tick for every listed run whose batten state is running with no cancel marker.
  A child that is absent, not running or covered by a pause is in no condition, as in batten.
- Checks, precedence (dead driver, usage limit, API error, quiet), episode keys, the quiet window, the API-error idle time and the idle clocks that subtract recorded sleeps and paused intervals move unchanged.
  A live wait marker still holds quiet, so a run waiting on a pull request never reads as a stall.
- `dobbyengine` declares its own reading types, since its import allowlist admits neither `battenshed` nor `shedengine`.
  The hub wiring in `internal/beamcli/dobbydeps.go` fills new Deps seams over the reads `internal/battencli/wire.go` already makes: one run's child by slug, the api-error revive, and the quiet window.
- The quiet window stays the Run-Shed `notice_quiet_min`, read as `notice_probe_s` already is.
  The driver-dead text names the respawn cap when batten's memory shows it reached, through an exported `battenshed` reader.
- The sent-notice marker, one episode key per line, moves to `notice-episode` in the run's durable directory under `_lym`; its path is declared in `internal/shedrun/paths.go`.
  Batten's status commit carries it beside the status and seed when it exists, and `lym batten cleanup` removes it with the run directory.
- Dobby's episode memory (the child state it last saw, when the episode began, the revive streak) rides in the run's entry of dobby's state record, so a re-exec keeps it.
- A notice that is not posted is retried on later probe ticks, at most three times, then dropped with a Warn, as batten dropped one.
- Cost: one child read per running run per probe tick, the same reads a pass made per probe.

## Known limits

- Between a marker write and the next batten transition the marker is an uncommitted change in the prime, as the status file already is during a watch.
  A machine switch in that gap can repeat one notice.
- Dobby's first sight of a running child after a machine switch dates the episode from that sight, so a driver-dead key the other machine sent can be sent once more.

## To check by hand

- The hub's deployed stencil copies are not edited here; the landing's deploy rewrites them from `contracts/`.
  After it, confirm no deployed rule or stencil still names the moved notices as `[batten notice]`.
- Run `lym gate test` over `./internal/dobbyengine`, `./internal/battenshed`, `./internal/battencli` and `./internal/beamcli`, and a live dobby against a real running child, since no test here sends a notice through a real orch.
