# Card 6 — startup-probe-backoff

**Edit:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `plan:internal/shuttleengine#dismissalGap`

**Intent:** `awaitStartup` probes from `pollInterval(cfg)` and backs off while the pane shows nothing new, so a provider that is ready within about a second is seen within about a second (decision D5 as superseded by the parent's backoff addendum under `PATTERN-daemon-wakeups`).
The interval starts at `pollInterval(cfg)`.
After each probe `awaitStartup` compares `run.lastStartupCapture` with the previous probe's, kept in a local previous-capture variable beside a seen flag so the first probe is always a change even when its capture is empty: a different capture, the first probe's included, sets the interval to the base; an equal one, or a failed capture that leaves the field unchanged, doubles it, capped at `dismissalGap(cfg)` from card 5.
The sleep after the probe is the interval so set; a ready pane ends the loop before any sleep, as today.
The tick cap is `startupTickCap(window, pollInterval(cfg))`, computed from the base: backoff only makes ticks fewer, so the base-computed count stays the bound, and under a clock that never advances the loop still runs exactly that many probes.
`startupTickCap`'s comment is reworded from "probe interval" to the base poll interval it now receives.
The `awaitStartup` probe-cadence comment is rewritten, one sentence per line: it names the base, the doubling while the capture is unchanged, the reset to the base on a changed capture, the ceiling at the dismissal gap, and that stray keypresses are bounded by card 5's gap and not by the probe cadence.
Each probe still runs `checkLivenessTick`, one reed status and, before ready, one pane capture; no sub-second literal is added.
The record's Constraints line saying `awaitStartup` gets no backoff and that `PATTERN-daemon-wakeups` is amended is replaced by the parent's later addendum, which this card follows: `PATTERN-daemon-wakeups` is not amended.
Bound: a pane that turns ready after a static boot screen is seen within one gap of turning ready, and Claude's boot redraws the pane, which resets the backoff; per-second probes stop at the first ready probe and never exceed the tick cap.
`doc.go`'s startup paragraph gains a sentence on the probe cadence.
Rows in `TestStartup_ReadyProbeTiming`, whose table gains a `captures []string` field queued on the fake reed, with its "probe cadence matches Wait" row replaced and its doc comment and `//testtiming:keep` reason reworded: an unchanged, empty capture doubles to the gap, the empty first capture counting as a change (`LivenessEveryNPolls` 4, five pending probes then ready, no captures queued) and elapses 1+2+4+4+4 seconds; a changed capture resets (`LivenessEveryNPolls` 10, captures `a`,`a`,`a`,`b` then ready) and elapses 1+2+4+1 seconds, the last one-second sleep being the acceptance row where a pane that turned ready after a changed capture is seen one poll interval later; the existing "ready on the first probe never sleeps" row stays.
`TestStartup_TickCap` computes its expected count as `startupTickCap(timeout, pollInterval(cfg))` without the `LivenessEveryNPolls` factor, and its "NoOutputFiles" subtest runs on a copy of `shortStartupConfig` with `LivenessEveryNPolls` 4, so a regression to the old multiplied interval changes the count and fails the row.

**ImpactSummary:** `awaitStartup` has one caller, `start`; every shuttle start now sleeps the base interval after a changed capture, tests on `fastConfig` (liveness every poll) see no cadence change, and the tick-cap test's expected count is the only other assertion that read the old interval.
