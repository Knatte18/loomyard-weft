---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-convergence-guard.md
    summary: Card 5's fixture enumeration covers only the review-side read and misses TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit, whose first row settles CONVERGED over a ledger with an open BLOCKING entry that the new guard refuses
---

### [MEDIUM] Card 5 misses a CONVERGED fixture with an open BLOCKING ledger entry

**Location:** `_lyx/plan/05-convergence-guard.md` (`**Edit:**` list, the fixture paragraph of `**Intent:**`, and `**ImpactSummary:**`)

**Issue:** Card 5 lists the fixtures the guard's new reads break, but only for the review-side check, which needs a parsing review.
It does not cover the ledger-side check.
`internal/shedadapters/bouncer_commit_test.go:112-131`, the first row of `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` ("findings are filtered, labelled and given their ledger rounds"), drives `judgeFakeShuttle(2, bouncerVerdictContent("CONVERGED"), evidenceLedgerContent(t, 2, tt.judged...), true)`.
Its judged ledger holds `{"beta", "open", "scope", "BLOCKING"}`, and the row expects `shedengine.Done` with a carry-over finding of severity `BLOCKING`.
Under D7 that judgment is refused and archived, so the row returns Stuck and fails `go test ./...` in card 5's own verify.
The test is in no card's targets.
Card 5's ImpactSummary says the only fixtures the guard rejects are the three verdict-less review helpers, which is incomplete.
I checked every other in-tree CONVERGED settle in `internal/shedadapters`, `internal/shedrecipe` and `internal/loomrecipe`.
Their ledgers are `ledger: []` or hold no open BLOCKING entry, and every round-2 ledger keeps its round-1 keys, so this row is the only one the ledger rules break.

**Fix:** Add `internal/shedadapters#TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` to card 5's `**Edit:**` list.
In `**Intent:**`, state that the row's open BLOCKING entry is changed to one the guard admits, for example `beta` at severity `MEDIUM` with the expected carry-over entry changed to match.
A CONVERGED settle can no longer carry an open BLOCKING finding.
Extend `**ImpactSummary:**` to name this test with the three review helpers.
Then go through the method once more for both of the guard's new reads, the review read and the ledger reads (open BLOCKING under CONVERGED, and a dropped round N-1 key), against every fixture that reaches a settle.
This finding repeats round 1's enumeration gap for a read that round did not cover, so it is filed as `design`.

## Focus notes

The focus directive's three questions, settled:

1. Card 6 (usage record): every fixture that writes a record with no review half and that a lens reader reads is targeted.
   These are the `writeRoundUsage` calls in `TestWriteRoundFacts_LensSection` (`bouncerfacts_test.go:324-370`) and the `usageOf` helper of `TestBurlerProducer_Call_ClusterExclusions` (`burler_test.go:652`).
   `TestBurlerProducer_Call_WritesRoundUsageRecord` reads `got.Review` and `got.Fix` as values (`burler_test.go:559`), so it breaks when they become pointers, and it is targeted.
   No package outside `internal/shedadapters` reads or writes the record, and the ImpactSummary names both test fixtures.
   Card 5 (guard): the review-side fixtures `bouncerReport`, `layoutBouncerRound1Report` and `newLoomBurler` are targeted and named.
   They cover every CONVERGED harvest in those three packages, since `layoutApprovedGeneration` writes `roundReviewPath` and not `ReportName`, and `layoutCirclingRun` is not a Bouncer settle.
   The ledger-side gap is F1.
2. Card 7: every pass whose result reaches a Call is recorded.
   Those passes are `runSeedSpawn`'s attach and `Run`, `judgeCall`'s attach and `Run`, and the two entry-time probes `awaitLiveJudge` and `awaitLiveSeed`.
   No other `Attach`, `AttachIfLive` or `Run` call exists in the Bouncer.
   Both probes carry the re-sign arrow with the new `(shuttleengine.Result, bool, error)` results.
   `runSeedSpawn` and `judgeCall` keep their signatures, so they need no arrow.
   Not a defect.
3. The planned tests do not duplicate an existing row.
   Card 4's new rows extend `TestBouncerEntry_ConstructionFailures`, which has no unknown-key or no-entry row today.
   The generic rejection of `configRejectUnknown` is pinned in `config_test.go`, while these rows pin `bouncerEntry`'s accepted-key list, which D5 changes.
   Card 5's `TestBouncer_JudgeCall_RefusedJudgment` covers the refusal route, which no existing test reaches, and it uses the `CONVERGED is Done` row of `TestBouncer_JudgeCall_Verdicts` as its pass-through case instead of re-pinning it.
   Not a defect.
   On the test budget, `internal/shedadapters` has 158 top-level tests against a row of 159.
   Card 5's one new test fits, and card 7 raises the row for its own.
