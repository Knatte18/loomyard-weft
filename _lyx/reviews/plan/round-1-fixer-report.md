# Plan-Review round 1 — fixer report

All four findings validated against the code and fixed.
`lyx loom validate-plan` passes after the fixes, with only the informational `create-new-unit` note for card 7's new test file.

## Fixed

### F1 (BLOCKING) — fixtures the tighter read rules reject

Confirmed: `burlerengine.ParseReview` rejects a review with no `verdict:` (`internal/burlerengine/verdict.go` `parseVerdict`), and `bouncerReport` (`internal/shedadapters/bouncer_judge_test.go`), `layoutBouncerRound1Report` (`internal/shedrecipe/entries_bouncer_test.go`) and `newLoomBurler` (`internal/loomrecipe/fixture_test.go`) all write verdict-less reviews that CONVERGED settles read.
Confirmed too that `TestBurlerProducer_Call_ClusterExclusions`' `usageOf` and `TestWriteRoundFacts_LensSection`'s existing subtests write records with no review half.
No loomcli test reaches a CONVERGED settle, so none was added there.

- Card 5: added `internal/shedadapters#bouncerReport`, `internal/shedrecipe#layoutBouncerRound1Report` and `internal/loomrecipe#newLoomBurler` under `**Edit:**`; the Intent states that each renders a review whose frontmatter holds `verdict: APPROVED` and `findings: []`, naming the tests that read each; the ImpactSummary names the newly rejected fixtures and their packages.
- Card 6: added `internal/shedadapters#TestBurlerProducer_Call_ClusterExclusions` under `**Edit:**`; the Intent states that `usageOf` and the existing `TestWriteRoundFacts_LensSection` subtests write a review half; the ImpactSummary names both.

### F2 (MEDIUM) — entry-time attach passes record no usage entry

Confirmed: `awaitLiveJudge` and `awaitLiveSeed` discard the `AttachIfLive` result, and `Bouncer.Call` is their only caller.
Card 7 now lists `Bouncer.Call` and re-signs both probes with the arrow to return `(shuttleengine.Result, bool, error)`; the Intent has `Call` record the attached pass, and `TestBouncer_UsageRecord` gains the entry-time judge attach and re-bounce seed attach rows.
The ImpactSummary names the probes and their one caller.

### F3 (LOW) — clean-CONVERGED row duplicates an existing test

Confirmed against `TestBouncer_JudgeCall_Verdicts`' `CONVERGED is Done` row.
Card 5 drops the row from `TestBouncer_JudgeCall_RefusedJudgment` and states that the existing row, over the fixed `bouncerReport`, pins the pass-through case.

### F4 (LOW) — construction refusal outside the construction-failure table

Card 4 moves the `model`/`effort`/`version` refusal rows into `TestBouncerEntry_ConstructionFailures` beside the no-entry row, and `TestBouncerEntry_EnvJudgeFallback` keeps only the per-spawn triples.

## Disputed

None.

## Deferred

None.
