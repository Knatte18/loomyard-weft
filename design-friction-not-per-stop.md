# Friction is gathered once per run, not by an agent session at every stop

`friction-not-per-stop` · Note · enhancement, shed, loom

## Seen (2026-10-19, run `ubuntu-2404-support`)

The run stopped many times in one evening (Webster card halt, Plan-Bouncer commit failure, the review re-run after a goto, the interrupted review).
At each stop the Go driver started a friction agent session (strand `<shortname>:<slug>:friction`), and often a tackler and an arbiter after it.
The friction session held the driver lock while it ran, so `lyx loom resume` was refused ("driver still holds its driver lock and is settling the stop") until it ended; one friction strand the operator stopped by hand stayed registered live and kept the step waiting until the orch removed the strand.
So a single wrong goto cost not only a review round but a friction session, plus tackler and arbiter sessions, per stop that followed.

## Operator view, not a decision (2026-10-19)

Friction does not need to run at every stop.

## Idea

Friction notes are gathered once, by the run's Friction-Reflect at the end, from what the stops left in the run's records; no agent session starts per stop for friction.
A stop never waits on a friction session before the run can be resumed.
