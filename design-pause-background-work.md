# A hub pause leaves background forks and shells running, so hibernate never confirms

`pause-background-work` · Note · bug, shuttle, webster

## Seen (2026-10-14)

`lyx hibernate request` with no trigger paused the hub at once, but its confirmation never passed: after the grace, `lyx tender status` still read `ly:overview-seats:webster` and `ly:overview-seats:driver` as `busy` with cause `background`, and the orch as `busy` with cause `turn` while it answered the operator.

- The pause interrupts a session's running turn, but work the session started in the background keeps going: Webster's implementer forks, and the driver's background wait loop.
- A session with background work reads busy, so confirmation withholds until its ten-minute bound and the request fails.
- The operator cancelled the request and hibernated by hand.

## Idea

- A hub pause also stops, or lets finish and then holds, each session's background work: forks, background shells, Monitors; or confirmation treats a session whose only background work is a known-safe wait (the driver's loop) as idle.
- What a paused fork should do (stop and be re-spawned on resume, or finish its batch and write its report) needs a design choice, since killing an implementer fork mid-batch leaves a half-done batch for Webster's recovery.
- The orch's own turn should not withhold confirmation when the orch itself requested the hibernate and ends its turn.
