---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/14-gate-quiescence.md
    summary: Card 14 extends TestPollEventsTick_ForkOutstandingKeepsWaitingPastBound for a gated run with an outstanding fork, which TestPollEventsTick_ShellExpiry already pins, against D7's extend-only-if-uncovered condition
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/08-accumulated-exclusions.md
    summary: Card 8's new burler_test table repeats the drop-with-warning behavior TestBurlerProducer_Call_ClusterExcludeDropWarning table-drives, and its round-2 union row repeats the ClusterExclude assertion kept in TestBurlerProducer_Call_ProfileCarriesDerivedFields
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/07-round-usage-record.md
    summary: Card 7's roundusage_test.go fanned round-trip duplicates the burler_test test the same card plans, which already writes a fanned record through doneExit and reads it back
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/11-env-row-fans.md
    summary: Card 11 plans a new entries_burler_test.go test whose cases are rows of TestBurlerRoundEntry_EnvReviewFallback and TestBurlerRoundEntry_ConstructionFailures
---

### [MEDIUM] Card 14 re-pins a gated fork wait that an existing row already covers

**Location:** `_lyx/plan/14-gate-quiescence.md`, its last Intent paragraph and the `internal/shuttleengine#TestPollEventsTick_ForkOutstandingKeepsWaitingPastBound` target.

**Issue:** D7 says a shuttle wait test pins the gated-fork wait rule, "extended if no existing test covers a gated run with an outstanding fork".
An existing test already covers it.
`TestPollEventsTick_ShellExpiry` in `internal/shuttleengine/waiting_test.go` has the row "a fork beside payload-reported shells keeps the gated run waiting" (`gated: true, touchOutput: true`, a `BackgroundFork` outstanding).
That row asserts the run is still waiting one second before the bound and that the tick at the bound returns no outcome and no hold.
`TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` already pins that a gated waiting turn end with its output files present never evaluates the gate (`gateCalls != 0` check).
Card 14 makes `TestPollEventsTick_ForkOutstandingKeepsWaitingPastBound` gated with its output present, so two tests would pin one behavior, which breaks `PATTERN-test-economy` and D7's condition.

**Fix:** Drop the `TestPollEventsTick_ForkOutstandingKeepsWaitingPastBound` target and that paragraph from card 14.
Name `TestPollEventsTick_ShellExpiry`'s fork row as the test D7 relies on.
If the card wants "gate never evaluated" stated on the fork case itself, add a gate-call count to that row's passing gate instead of making a second test gated.

### [LOW] Card 8's new table overlaps two existing producer tests

**Location:** `_lyx/plan/08-accumulated-exclusions.md`, the `burler_test.go` paragraph.

**Issue:** The new table's rows for round 1 dropping a focus file's excludes with a warning, a record naming another fan or no fan dropping them with a warning, and an unrun lens being dropped all test one behavior: focus-file excludes that Go drops never reach `ClusterExclude`, and a warning names them.
`TestBurlerProducer_Call_ClusterExcludeDropWarning` already table-drives that behavior, asserting `ClusterExclude` and the presence of the drop warning, for the fan-less case.
`TestBurlerProducer_Call_ProfileCarriesDerivedFields` keeps its `ClusterExclude == [lensA]` assertion for a round-2 focus file, and the card only adds a usage record to keep it eligible.
The new table's union of rounds 2 to N then pins the same round-2 assertion a second time.
Card 8 keeps `excludedLenses` free of a direct test, which settles the focus question about a `focus_test.go` duplicate, but the producer level now has two overlaps.

**Fix:** Grow `TestBurlerProducer_Call_ClusterExcludeDropWarning` into the one exclusion table at the producer surface.
Its rows cover the fan-less drop, round 1's drop, the ineligible-record drops and the unrun lens, plus the union and held-set rows, with each row's wanted `ClusterExclude`, `ClusterExcludeHeld` and warning.
Remove the `ClusterExclude` assertion and the focus file from `TestBurlerProducer_Call_ProfileCarriesDerivedFields`, so that test keeps only the derived paths and the marker.

### [LOW] Card 7 plans the fanned record round-trip twice

**Location:** `_lyx/plan/07-round-usage-record.md`, the test paragraph.

**Issue:** `roundusage_test.go` round-trips a fanned and a solo record through `writeRoundUsage` and `readRoundUsage`.
The same card's `burler_test.go` test checks that a successful fanned round leaves a record with the fan, models and lenses.
That test writes the record through `doneExit` and reads it back, so it already round-trips a fanned record.
Only the solo record and the unknown-token rendering are covered in `roundusage_test.go` alone.

**Fix:** Keep in `roundusage_test.go` only what the producer test does not reach: the unknown-token and unknown-wall-time rendering of the file, and the solo record's empty fan and nil lenses.
Or add a solo row to the `burler_test.go` test and leave `roundusage_test.go` to the rendering alone.

### [LOW] Card 11's new test is rows of two existing tests

**Location:** `_lyx/plan/11-env-row-fans.md`, the `entries_burler_test.go` paragraph.

**Issue:** The new test checks that an env fan reaches the profile, that a row absent from the map keeps the recipe's value, and that an env fan beside a literal `cluster-fan` fails to build naming both.
`TestBurlerRoundEntry_EnvReviewFallback` already pins the row-keyed Env-map rule for `RowReviewModels`, including its `RowMapEntryReplacesEnvModelsForItsOwnRowOnly` subtest, where a row's entry applies to its own row only.
The fan map follows the same rule.
`TestBurlerRoundEntry_ConstructionFailures` is where a `burlerRoundEntry` build failure is pinned.

**Fix:** Add the env-fan and absent-row cases as a subtest of `TestBurlerRoundEntry_EnvReviewFallback`, and the conflicting-sources case as a row of `TestBurlerRoundEntry_ConstructionFailures`, with no new test function.

## Focus verdicts

- Card 6: no defect.
  It plans no new test.
  `TestEngine_Run_ShuttleOutcomes` gains assertions on each half's times and usage, and `TestEngine_Run_ForkSubagentsSpecWiring` gains a `wantLenses` column, as the focus asks.
- Card 8: `excludedLenses` gets no direct test, so the union and eligibility rules sit at one level.
  The remaining overlap with existing producer tests is F2.
- Card 5: no defect.
  The `usage_test.go` cases are rows of one table-driven `SessionUsage` test.
  Its temporary-`HOME` fixture matches `TestAuditForks_FullLayout`'s precedent.
- Changed test plans: of the changed cards, only card 8 has a defect (F2).
  F1, F3 and F4 are on cards outside the three the focus names.
  They are recorded because the rubric's test-economy item binds every card.
