## Changes

- F1: `settleFresh` now saves the strand and writes the marker through `recordStartedArbiter`; when either write fails after `Shuttle.Start`, it calls `run.Stop()` before returning the error, appending a failed stop to it.
  The marker-write failure case is a row in `TestArbiterVerb_VerdictOutcomes` (new `wantError` field), not a new top-level test, to stay inside the `internal/loomcli` test budget.
- F2: the `circling.go` package comment lost the stray `// //` and no longer attributes the Bouncer-escalation bound to the arbiter's eligibility; it says the verbs refuse unless the run awaits at a Bouncer row with a pending escalation.
- F3: restored the two spaces before the comment on `parent_review_wait_min` in `template.yaml`.

`lyx gate test` over `./internal/loomcli`, `./internal/loomengine` and `./cmd/lyx` is green.

## Disputed

None.

## Deferred

None.
