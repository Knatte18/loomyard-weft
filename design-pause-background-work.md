# A hub pause leaves background forks and shells running, so hibernate never confirms

`pause-background-work` · Note · enhancement, shuttle, webster

## Seen (2026-10-14)

`lyx hibernate request` with no trigger paused the hub at once, but its confirmation never passed: after the grace, `lyx tender status` still read `ly:overview-seats:webster` and `ly:overview-seats:driver` as `busy` with cause `background`, and the orch as `busy` with cause `turn` while it answered the operator.

- The pause interrupts a session's running turn, but work the session started in the background keeps going: Webster's implementer forks, and the driver's background wait loop.
- A session with background work reads busy, so confirmation withholds until its ten-minute bound and the request fails.
- The operator cancelled the request and hibernated by hand.

## Idea

- A hub pause also stops, or lets finish and then holds, each session's background work: forks, background shells, Monitors; or confirmation treats a session whose only background work is a known-safe wait (the driver's loop) as idle.
- What a paused fork should do (stop and be re-spawned on resume, or finish its batch and write its report) needs a design choice, since killing an implementer fork mid-batch leaves a half-done batch for Webster's recovery.
- The orch's own turn should not withhold confirmation when the orch itself requested the hibernate and ends its turn.

## Operator view (2026-10-15, not a decision)

- Claude Code can pause a fork: the fork is picked in the terminal's fork/agent picker and stopped there, and the master is then told the fork stopped. A hub pause would have to reach each fork that way.
- Forks are not interrupted as a rule: Webster forks can live long (one ran 45 minutes on 2026-10-14), and cutting one off mid-batch is out of the question as a default.
- But that 45-minute fork held 300k+ tokens of context: its batch was too long, and interrupting and restarting it with a fresh context might have been cheaper.
  This points at a separate idea: a context bound on a Webster fork that ends it and restarts the rest of the batch fresh, which also gives a pause a clean place to stop.

