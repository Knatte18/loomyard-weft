---
round: 1
focus:
  - "In internal/gatecli/run.go and build.go, trace WaitStart and Start on every path that prints a tier summary line (shared build run inside the tier's slots, blocked tier, carried tier, rerun, --copies, empty scope, inherited or unslotted call): does SlotWaitSeconds equal the slot wait alone in each, as the help, package doc, doc comments and change description say, and does wait plus wall account for the time as a caller would read it?"
  - "The summary-line patterns in internal/gateslot/runlog.go now require the wait field: can a reader (FailedTier in internal/landingshed/verifygate.go, TierDiscovered in redfirstdotnet.go) meet a line or stored log written by a different lym binary, and does the change description's same-binary claim and its Merge danger door and blast radius hold?"
  - "Search for tests and fixtures that spell the summary line literally, such as internal/gateslot/startedlines_test.go: do they still represent the line the gate prints?"
  - "internal/gatecli/cli.go drops the --copies paragraph from the verb's help to fit a byte cap: is every fact it carried still stated by the flag help or elsewhere a caller of the help sees, and does the answer key license the removal?"
  - "Test coverage and economy of TestTierSummaryLine: do its cases cover the paths where the wait figure is derived differently, and does it duplicate what the existing FailedTier and TierDiscovered tests already hold?"
---

Seed focus for round 1, drawn from the committed diff and the board entry, which asks that a gate call show the slot wait and the run's wall time as separate values.
