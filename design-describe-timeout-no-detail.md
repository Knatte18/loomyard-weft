# Describe halts failed on an uninformative shuttle run outcome timeout

`describe-timeout-no-detail` · Note · bug, landing, shuttle

Imported from [issue #244](https://github.com/Knatte18/loomyard/issues/244).

The Describe step halted the run as `failed` with anomaly `producer-hard-failure` and the message `shedadapters: Describe (seats): shuttle run outcome timeout`.

Why it is abnormal:
- The note carries no detail on which timeout elapsed, how long the shuttle run had been going, or whether the agent was still working.
- A timeout on a seated shuttle run is classed as a hard failure, so the run stopped as `failed` rather than being resumed or escalated.
- The next attempt of the same step completed (`Describe / done`) about 34 minutes later, which suggests the timeout was not a defect in the producer's work.

What to do:
- Include the elapsed time, the timeout limit and the shuttle run's last known state in the halt reason.
- Decide whether a shuttle outcome timeout should be a retryable or `blocked` stop instead of `failed`.

Provenance: task `retire-d9`; note loom-halt.md.
