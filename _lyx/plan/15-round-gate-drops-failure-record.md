# Card 15 — round-gate-drops-failure-record

**Edit:**
- `internal/loomshed#NewVerifyGate`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#TestReadRubric`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `internal/shellguard#TestRows_PatternsDenyAndAllow`

**Delete:**
- `internal/loomshed/publishfailure_integration_test.go`
- `internal/loomshed#TestPublishFailure`
- `internal/loomshed#TestPublishFailureNote`
- `internal/loomshed#hexA`
- `internal/loomshed#hexB`
- `internal/loomshed#tracePlan`
- `internal/loomshed#failingGateTiers`
- `internal/loomshed#writeRecord`
- `internal/loomshed#fullObjectName`
- `internal/loomshed#failedTestPath`
- `internal/loomshed#failedTestPackage`
- `internal/loomshed#tracedLocation`
- `internal/loomshed#failingGate`
- `internal/loomshed#checkedPublishFailure`
- `internal/loomshed#checkedTracedCards`
- `internal/loomshed#promptSafeGlyph`
- `internal/loomshed#checkedLocations`
- `internal/loomshed#checkedCommitRange`
- `internal/loomshed#isMergeCommit`
- `internal/loomshed#publishFailureSteps`
- `internal/loomshed#PublishFailureNote`
- `internal/loomshed#writeFailureTrace`
- `internal/loomshed#parseTracePlan`
- `internal/loomshed#liveCard`
- `internal/loomshed#firstIntentLine`
- `internal/loomshed#codeSpans`
- `internal/loomshed#anchoredRunPattern`

**Extends:**
- `internal/loomshed/doc.go`
- `internal/shedadapters/doc.go`

**Intent:** Remove the round gate's read of the failure record and the review's `publish_failure` marker with the seam that fills it (D5, D11): a landing failure is the fixer's brief now, read by no review row.
`NewVerifyGate` no longer confirms a Publish failure: the step that prefixed the failing tests and appended a package pass of the record's tier goes, and the gate runs its told or narrowed command alone.
The record checks and the note behind them go with it, and only `WebsterRecordNote` stays in `publishfailure.go`; the tests of the removed functions and their fixtures go, and `TestVerifyGate_Scenario` drops its failure-record rows.
`RubricNotes` drops `PublishFailure`, `ReadRubric` stops filling a `publish_failure` marker, `BouncerConfig` drops its `PublishFailure` seam, and `Bouncer.rubricNotes` stops reading it; `shedrecipe.Env` drops `PublishFailure`, and `bouncerEntry` and `burlerReviewEntry` stop passing it.
`wire` stops filling it, and `envkit.nilLegal` drops its entry.
The Webster-Review rubric loses its `## Publish failure` section and step 5 of its review range, which widened the range by the failure record's merge-in commit; `TestReadRubric` drops its `publish_failure` rows.

`TestRows_PatternsDenyAndAllow`'s row whose stated reason is the form the publish-failure steps render keeps its command and rewords its reason to the tmux-tier command it pins, since those steps go.

`loomshed`'s and `shedadapters`' `doc.go` drop their failure-record and marker paragraphs.

**ImpactSummary:** One marker, one Env seam and the round gate's confirm step go; their readers are the Webster-Review rubric and the review rows' builders, all listed.
