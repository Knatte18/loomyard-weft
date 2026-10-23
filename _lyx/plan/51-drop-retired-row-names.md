# Card 51 — drop-retired-row-names

**Delete:**
- `internal/loomshed#NamePublish`
- `internal/loomshed#NamePRAlign`
- `internal/loomshed#NamePRGate`
- `internal/loomshed#NamePRRework`
- `internal/loomshed#NameMergeBouncer`
- `internal/loomshed#NameMergeBurlerReview`
- `internal/loomshed#NameMergeBurlerFix`
- `internal/loomshed#NameWebsterBouncer`
- `internal/loomshed#NameWebsterBurlerReview`
- `internal/loomshed#NameWebsterBurlerFix`
- `internal/loomcli#TestGoto_AwaitingAtPRGateMovesBackToPublish`

**Edit:**
- `internal/loomcli#newArbiterFixture`
- `internal/loomcli#TestArbiterVerb_VerdictOutcomes`
- `internal/loomcli#TestCirclingVerb_Refusals`
- `internal/loomcli#TestLatestDarnOutcome`
- `internal/loomcli#TestLoomPostRun_HaltNotes`
- `internal/loomcli#TestRecipeRouting`
- `internal/loomcli#TestResumeVerb`
- `internal/loomcli#TestLoopLockExit`

**Intent:** Delete the `loomshed` row-name constants of the rows the recipes no longer hold (D1): the Publish, PR-Align, PR-Gate, PR-Rework, Merge-Review and Webster-Review rows.
Every production reader moved off them in the earlier cards, the report onto its own literals for old runs; the tests that still name them switch to the current rows: the arbiter fixture and verdict test halt at Finalize or Merge-In, the circling refusal test names Code-Bouncer, the darn outcome, halt-note and routing tests name the new darn tail, `TestResumeVerb`'s own-leftover row names Code-Bouncer, a row that never merges, `TestLoopLockExit` drops its two PR-Gate awaiting rows, whose release its awaiting-at-Describe row already pins, and the goto test of an awaiting PR-Gate goes with the row.

**ImpactSummary:** Ten constants go; their remaining references are the tests listed here, all in loomcli.
