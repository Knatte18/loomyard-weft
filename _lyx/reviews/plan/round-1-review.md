---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/06-burler-half-usage.md:17
    summary: Card 6 plans a new engine_test.go test for halfOf passthrough and Result.Lenses, behaviour TestEngine_Run_ShuttleOutcomes and TestEngine_Run_ForkSubagentsSpecWiring already cover and should extend (PATTERN-test-economy)
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/08-accumulated-exclusions.md:31
    summary: Card 8 pins the same accumulation and eligibility rules twice, in a new burler_test.go table test and in a direct focus_test.go test of excludedLenses (PATTERN-test-economy)
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-claude-session-usage.md:25
    summary: Card 5 plans plural tests for the cases of one behaviour (Claude.SessionUsage reading) where one table-driven test is the rule
---

### [BLOCKING] Card 6 adds a test for behaviour existing engine tests already cover

**Location:** `_lyx/plan/06-burler-half-usage.md:17`

**Issue:** The card plans a new `engine_test.go` test asserting that a done round's `Result` carries both halves' usage and times "as the fake shuttle reported them", plus `Result.Lenses` for a cluster round and nil for a solo one.
The usage and times reach `Half` through `halfOf`, which copies the half's terminal `shuttleengine.Result`.
`TestEngine_Run_ShuttleOutcomes` (`internal/burlerengine/engine_test.go:598`) already pins that behaviour: its doc says "each half's identities and kept RunDir pass through to the Result unchanged", and it asserts `got.Review.SessionID`, `StrandGUID` and `RunDir` (`engine_test.go:720-727`).
The new fields are more of the same passthrough, so they are assertions in that test, not a new test.
`Result.Lenses` is the cluster-versus-plain distinction `TestEngine_Run_ForkSubagentsSpecWiring` (`engine_test.go:486`) already table-drives with a `"cluster profile"` row and a `"plain profile"` row, each running a done round.
Which lenses survive a `ClusterExclude` is already pinned at `clusterLenses` by `TestProfileValidate_ClusterExclude`, which card 3 extends;
`decideRound` only copies them out.
`PATTERN-test-economy` requires a test to cover something no existing test covers, else extend the existing one, so this is a constraint violation.

**Fix:** Replace the new test with two extensions.
`TestEngine_Run_ShuttleOutcomes` asserts that each half's `StartedAt`, `EndedAt` and `Usage` match the fake shuttle's scripted `Result`.
`TestEngine_Run_ForkSubagentsSpecWiring` gains a `wantLenses` column: the resolved lens names for the cluster row and nil for the plain row.
Name both test glyphs under the card's `**Edit:**` in place of the bare `engine_test.go` path.

### [BLOCKING] Card 8 tests the accumulation rules twice

**Location:** `_lyx/plan/08-accumulated-exclusions.md:31-32`

**Issue:** The card plans a new `burler_test.go` table test over `BurlerProducer.Call` covering the union of rounds 2 to N, the held set, a record naming another fan or no fan, and a lens the latest round did not run.
It also plans a `focus_test.go` test of `excludedLenses` "directly over a run directory with several focus files and records, including a missing record".
Both pin the same rules: the union across focus files, eligibility from the previous round's usage record, and dropping ineligible entries.
They differ only in the missing-record case, which is one more eligibility row.
One behaviour pinned by two tests at two levels is a redundant test, which `PATTERN-test-economy` and the rubric's test-economy item treat as a defect.

**Fix:** Keep one test.
The preferred choice is the `burler_test.go` table at the producer surface, with a row for a missing usage record making the previous round's entries ineligible.
Then drop the `focus_test.go` case and the `focus_test.go` target.
If the card instead keeps the direct `excludedLenses` test for the union and eligibility rules, the `burler_test.go` table covers only `Call`'s own wiring: round 1 dropping excludes, and `ClusterExclude` and `ClusterExcludeHeld` being filled from `round` and `round-1`.

### [LOW] Card 5 plans several tests for one behaviour

**Location:** `_lyx/plan/05-claude-session-usage.md:25`

**Issue:** "`usage_test.go` gains tests over fixture transcripts" lists four cases of the one behaviour `Claude.SessionUsage` provides: repeated message lines counted once, forks summed and broken out, no `subagents/` directory, and an unreadable transcript reading unknown.
"Tests" in the plural invites one function per case, while `PATTERN-test-economy` makes the cases of one behaviour rows of one table-driven test.

**Fix:** State that `usage_test.go` gains one table-driven `SessionUsage` test whose rows are the four cases.

## Focus verdicts

Each directive was reached, and these items were settled as not defects:

- **Fidelity:** every decision from D1 to D9 and every Constraint is carried.
  D7 is carried by card 14: the rewritten guard, the failure message naming the new ground, and the gated fork-wait extension.
  No existing gated test in `waiting_test.go` has an outstanding fork, so the extension is licensed.
  D9 is carried by cards 13 and 14: no Webster row in `RowClusterFans`, no literal `cluster-fan` on any row, and the corrected recipe comment.
  No card adds unlicensed scope.
  Card 14's edit to the doc comment of `TestWebsterRoundCarriesNoGateKey` removes a premise that D9 makes false.
  Card 8 ties eligibility to the previous record's fan name.
  That reading is the one the acceptance criterion settles ("including every lens after a switch from solo or from another fan, is dropped").
- **Granularity:** the new unexported members of cards 5, 7, 8 and 9 each serve one surface and sit in that surface's card.
  Card 4's `SessionUsage` and `UsageReader` are one capability, since the interface's method returns the type.
- **Member glyphs and re-sign arrows:** `applyClusterExclude` (card 3) and `writeRoundFacts` (card 9) change signature and carry the arrow.
  The other named members keep their signatures: `Run.finalize`, `halfOf`, `decideRound`, `BurlerProducer.Call`, `runSeedSpawn`, `burlerRoundEntry`, `bouncerEntry`, `LoadConfig`, `ResolveReview` and `loomCLI.wire`.
- **Card order:** every handle a card uses is declared on an earlier card.
- **Attack surface:** cards 1, 3, 8, 11 and 12 state their bounds.
  Card 10 adds no edge: it removes the seed's ask and adds rules to the judge's ask.
- **Verify:** `./...` under `go test` and `go test -tags integration` covers every targeted package, and the section's order matches the plan template.
