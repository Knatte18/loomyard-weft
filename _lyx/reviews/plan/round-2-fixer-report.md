# Plan-Review round 2 — fixer report

Every finding was validated against the code and fixed.
`lyx loom validate-plan` passes over the fixed plan.

## Fixed

- **F1** (`14-gate-quiescence.md`): confirmed that `TestPollEventsTick_ShellExpiry`'s row "a fork beside payload-reported shells keeps the gated run waiting" (`internal/shuttleengine/waiting_test.go`) is gated, has its output file present and a passing gate, and still waits at the shell bound.
  The card drops the `TestPollEventsTick_ForkOutstandingKeepsWaitingPastBound` target, and its closing paragraph now names that row as the test D7's shuttle wait rule rests on, with no shuttle test change.
- **F2** (`08-accumulated-exclusions.md`): the card now grows `TestBurlerProducer_Call_ClusterExcludeDropWarning` into the one producer-surface exclusion table, renamed `TestBurlerProducer_Call_ClusterExclusions`, whose rows state the wanted `ClusterExclude`, `ClusterExcludeHeld` and drop warning.
  `TestBurlerProducer_Call_ProfileCarriesDerivedFields` loses its focus file and `ClusterExclude` assertion, keeping the derived paths and the marker.
  The card plans no new test function.
- **F3** (`07-round-usage-record.md`): dropped the `roundusage_test.go` Create target.
  The card's one `burler_test.go` table now covers the fanned record, the solo record's empty fan and nil lenses, the unknown token and wall-time rendering, the gate-failed round and the unwritable record, all read back through `readRoundUsage`.
- **F4** (`11-env-row-fans.md`): replaced the `entries_burler_test.go` file target with `TestBurlerRoundEntry_EnvReviewFallback` and `TestBurlerRoundEntry_ConstructionFailures`.
  The env-fan and absent-row cases become a subtest of the first, and the conflicting-sources case becomes a row of the second.

## Disputed

None.

## Deferred

None.
