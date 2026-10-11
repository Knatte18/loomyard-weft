---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-convergence-guard.md, _lyx/plan/06-usage-record-merge.md
    summary: Cards 5 and 6 tighten a read rule without listing the existing test fixtures the tighter rule rejects, so the plan's own verify fails in shedadapters, shedrecipe and loomrecipe
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-bouncer-usage-entries.md
    summary: Card 7 records no entry for a judge or seed pass harvested through the entry-time attach probes awaitLiveJudge and awaitLiveSeed, though the pass's result reaches that Call, against D10
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-convergence-guard.md
    summary: The clean-CONVERGED row of TestBouncer_JudgeCall_RefusedJudgment duplicates the CONVERGED is Done row of TestBouncer_JudgeCall_Verdicts
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-per-segment-bouncer-threading.md
    summary: Card 4 puts the refusal of a row naming model, effort or version into TestBouncerEntry_EnvJudgeFallback instead of the construction-failure table TestBouncerEntry_ConstructionFailures
---

### [BLOCKING] Cards 5 and 6 leave the fixtures their tighter read rules reject untargeted

**Location:** `_lyx/plan/05-convergence-guard.md`, `_lyx/plan/06-usage-record-merge.md`

**Issue:** Both cards make a read stricter and neither enumerates the existing fixtures that the stricter read now rejects.

Card 5: the guard refuses a CONVERGED whose round review does not parse through `burlerengine.ParseReview`.
`ParseReview` requires `verdict:` to be exactly `APPROVED` or `BLOCKING` (`internal/burlerengine/verdict.go:226`).
The review fixtures every existing CONVERGED settle reads carry no verdict:

- `internal/shedadapters/bouncer_judge_test.go:65` `bouncerReport` renders `---\nround: N\n---\n...`, the report used by `TestBouncer_JudgeCall_Verdicts`, `TestBouncer_JudgeCall_PreviousLedgerHandling`, the clear tests (`bouncer_clear_test.go`), the commit tests (`bouncer_commit_test.go:76`) and others;
- `internal/shedrecipe/entries_bouncer_test.go:467` `layoutBouncerRound1Report` writes `a review\n`, read by `TestBouncerEntry_CommitSeam`, `TestBouncerEntry_CarryOver` and `TestBouncerEntry_ApproveSeam`, which expect a Done;
- `internal/loomrecipe/fixture_test.go:57` `newLoomBurler` writes `review` as every round's review, which the loomrecipe sequence tests' CONVERGED judges settle over.

After card 5, each of these CONVERGED judgments is archived and re-judged until the bounce budget blocks the run, so those tests fail.
Card 5 lists none of these fixtures, and `internal/loomrecipe` is not touched by any card.
Its ImpactSummary ("a judgment that passes today's parse checks and the new rules settles exactly as before") misses this blast radius: today's parse checks never read the review, so every in-tree CONVERGED fixture fails the new rule.

Card 6: `roundUsage.Review` becomes `*halfUsage`, and `readCompletedRoundUsage` reads a record with no review half as missing.
Existing fixtures write records with no review half:

- `TestBurlerProducer_Call_ClusterExclusions` (`internal/shedadapters/burler_test.go:652`, `usageOf` builds `roundUsage{Fan, Lenses}`): its union and held-set rows expect exclusions that now read as ineligible.
  Card 6 does not list this test.
- `TestWriteRoundFacts_LensSection` (`internal/shedadapters/bouncerfacts_test.go:324`, `:327`, `:369`): its existing rows expect `Fan:` and `Lenses run:` lines that now render as an unreadable record.
  Card 6 lists the test but its Intent only adds a case, never updating these rows.

Card 6's ImpactSummary ("every other reader is a test") names the test readers but not that they break.

**Fix:** In card 5, add the fixture members to `**Edit:**` (`internal/shedadapters#bouncerReport`, `internal/shedrecipe#layoutBouncerRound1Report`, `internal/loomrecipe#newLoomBurler`, plus any other review writer a CONVERGED settle reads, such as the inline `a review\n` write in `TestBouncerEntry_ReportNamePinning` if it reaches a settle).
State in the Intent that each one renders a review that parses with no BLOCKING finding, for example `verdict: APPROVED` and `findings: []`.
Rewrite the ImpactSummary so it names the fixtures and packages that the guard newly rejects.
In card 6, add `internal/shedadapters#TestBurlerProducer_Call_ClusterExclusions` to `**Edit:**` and state in the Intent that `usageOf` and the `TestWriteRoundFacts_LensSection` rows write a review half.
Name both in the ImpactSummary.

### [MEDIUM] Card 7 drops the usage entry of a pass harvested by an entry-time attach

**Location:** `_lyx/plan/07-bouncer-usage-entries.md`

**Issue:** D10 records an entry "for every pass whose shuttle call returned a result", and the acceptance criterion counts "one `judge` entry per judge pass whose result reached a Call".
Card 7 calls `recordPass` only from `runSeedSpawn` and `judgeCall`.
`Bouncer.Call` has two more attach sites: the entry-time judge probe `awaitLiveJudge` (`internal/shedadapters/bouncer.go:464`), which `Call` harvests through `settle` at line 340, and the re-bounce seed probe `awaitLiveSeed` (line 491).
Each gets a `shuttleengine.Result` from `AttachIfLive` and discards it ("deliberately carrying no shuttleengine.Result out").
A judge pass harvested after a driver crash therefore lands in no `judge` list, though its result reached this Call.
D10's carve-out is narrower: it skips only a judgment "a crashed Call never harvested that a later Call settles from disk".

**Fix:** Have `awaitLiveJudge` and `awaitLiveSeed` return the attached result, re-signed with the arrow, for example `` `internal/shedadapters#Bouncer.awaitLiveJudge` -> `func (b *Bouncer) awaitLiveJudge(round int) (shuttleengine.Result, bool, error)` ``.
Have `Call` record the pass on an attach, listing `internal/shedadapters#Bouncer.Call` under card 7's `**Edit:**`.
Add a row to `TestBouncer_UsageRecord` for a judge harvested by the entry-time attach.

### [LOW] Card 5's clean-CONVERGED row duplicates an existing test

**Location:** `_lyx/plan/05-convergence-guard.md`

**Issue:** `TestBouncer_JudgeCall_RefusedJudgment` plans the row "a clean CONVERGED, which settles Done exactly as before".
`TestBouncer_JudgeCall_Verdicts` (`internal/shedadapters/bouncer_judge_test.go:246`) already pins a CONVERGED harvest settling Done with the ledger pointer.
Once F1's fixture fix gives it a parsing review with no BLOCKING finding, that row is the pass-through case.

**Fix:** Drop the row from the new table and rely on `TestBouncer_JudgeCall_Verdicts`, or put the pass-through assertion there.

### [LOW] Card 4 places a construction refusal outside the construction-failure table

**Location:** `_lyx/plan/04-per-segment-bouncer-threading.md`

**Issue:** Card 4 rewrites `TestBouncerEntry_EnvJudgeFallback` to also assert that a row naming `model`, `effort` or `version` is refused.
That is a construction failure, and `TestBouncerEntry_ConstructionFailures` is the table for those; card 4 already adds its no-entry row there.
`PATTERN-test-economy` puts many items of one kind into one table.

**Fix:** Add the three retired-key rows to `TestBouncerEntry_ConstructionFailures`, and keep `TestBouncerEntry_EnvJudgeFallback` to the seed and judge triple each spawn takes.

## Verdicts on the focus items

- Card 4 granularity: it is one reviewable unit.
  `JudgeSettings` losing `Model`/`Effort`/`Version` breaks `loomCLI.wire` (`internal/loomcli/wiring.go:767`), which writes `Env.JudgeModel`, so resolution and threading cannot land apart without an interim mapping.
  Card 3's interim `bouncerEntry` edit is forced by compilation: `BouncerConfig` loses `Model`, and `Env.RowBouncerModels` needs card 3's `shedadapters.BouncerModels`, so the order cannot avoid it.
- D7's clear: an existing path does it.
  `settledGeneration` (`internal/shedadapters/bouncer.go:439`) archives a CONVERGED round at Call entry before the replay branch, so card 5's statement holds.
- D8 at entry: card 5 runs the key check on the replay of a CONTINUE or CIRCLING round.
  It skips the check for a round with an escalation record, but `escalate` writes that record only after a harvest that has already passed the guard, so nothing settles unchecked.
- `Env.RowBouncerModels` keyed by row name: not a defect.
  `Constructor` is `func(name string, cfg Config, env Env)` (`internal/shedrecipe/recipe.go:31`) and is never told the row's `segment`, and `RowReviewModels` and `RowClusterFans` are keyed by row name.
  Each segment's values still reach only that segment's row.
- Card order: no card uses a symbol a later card creates.
  `mergeRoundUsage` (card 6) precedes `recordPass` (card 7), and card 7's re-judge row relies on card 5's guard, which comes earlier.
- Test budget: `internal/shedadapters` has 158 top-level tests against a row of 159.
  Card 5's one new test fits; card 7's is the one that exceeds the row, and card 7 lists the file.
  `internal/loomengine` sits at its 41 and gains no top-level test.
- Re-sign arrow: no card changes a function's parameters, results or receiver.
  `newRoundUsage`, `ResolveJudge`, `ReworkSpec`, `newBouncerFixture` and `newTestEnv` keep their signatures, and the rest are struct field changes.
- ImpactSummary counts: card 4's one writer (`loomCLI.wire`) and one reader (`bouncerEntry`) of the judge fields match the tree.
  So do card 6's two production readers of the usage record (`computeLensFacts`, `lensesRunUnderFan`).
  Card 5's and card 6's blast radii are wrong, as F1 says.
- Attack surface: card 4 and card 5 state what their changes let through or skip and what bounds them.
- `## verify:` covers every targeted package through `./...`, including `cmd/lyx` and `internal/testkit/envkit`, in the template's order.
