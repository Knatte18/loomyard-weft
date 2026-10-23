# Card 13 — darn-write-no-reroute

**Edit:**
- `internal/loomshed#DarnDeps`
- `internal/loomshed#DarnTold`
- `internal/loomshed#darnWrite.Call`
- `internal/loomshed#darnWrite.checkSeams`
- `internal/loomshed#darnResumeWayForward`
- `internal/loomshed#renderPriorWork` -> `func renderPriorWork(latest DarnOutcome, found bool) string`
- `internal/loomshed#newDarnFixture`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/loomshed#TestRenderPriorWork`
- `internal/darnengine#DarnInputs`
- `internal/darnengine#DarnTable`
- `internal/darnengine#newSpecInputs`
- `internal/darnengine#TestDarnSpec`
- `contracts/stencils/darn/darn-template-write.md`
- `internal/loomcli#darnDeps`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/shedrecipe#darnTestEnv`
- `internal/loomshed#darnFixture`

**Delete:**
- `internal/loomshed#darnWrite.rerouteReason`
- `internal/loomshed#rerouteReady`
- `internal/loomshed#writerRunsFromReady`
- `internal/loomshed#darnRunStart`
- `internal/loomshed#darnRecordWrittenAt`
- `internal/loomshed#darnRecordHead`
- `internal/loomshed#darnReadyTip`
- `internal/loomshed#darnRecordedWork`
- `internal/loomcli#TestDarnDeps_PublishFailureNamesThePresentRecord`
- `internal/loomcli#TestDarnDeps_RejectionSeamsReadAndClearTheRecord`

**Extends:**
- `internal/loomshed/doc.go`

**Intent:** Darn-Write loses its reroute on a failure record and its pending-rejection read (D9, D11): darn fixes its own failures inside its rows, and no PR-Gate reject reaches the writer any more.
`DarnDeps` drops `ReadRejection`, `ClearRejection`, `RecordRejection`, `PublishFailure`, `RunStart`, `TaskHead`, `Dirty`, `ParentTips` and `IsAncestor`, keeping `Commit` and `LatestOutcome`; `checkSeams` checks the two left.
`darnWrite.Call` no longer reads a rejection or the failure record and never returns a rerouting Stuck: it renders the prior-work note from the latest outcome alone, runs the session, commits, and appends the resume way forward to a gate-failed Stuck as today.
`darnResumeWayForward` drops its clause about taking the parent in through Merge-In, since the run's Merge-In is gone from darn.
`DarnTold` drops `RejectionFindings`; `renderPriorWork` drops the failure-record path argument and its sentence about the last Publish.
`darnengine.DarnInputs` drops `RejectionFindings`, `DarnTable` drops the `rejection_findings` marker, and the darn writer stencil drops that marker and its lead-in sentence; `wireDarn` stops telling it.
`loomcli.darnDeps` fills only the two seams left, so the loomcli tests of the removed seams go; `envkit.nilLegal` drops `Darn.RecordRejection`, `envkit.FullEnv` fills only the two seams, and `darnWriteEntry`'s required-seam list and its construction-failure test shrink to them, and loomcli's `intentionallyNil` drops the removed `Darn.*` seams.

`TestDarnWrite_Call` loses its rejection and reroute rows and keeps the prior-work, commit and gate-failure rows; `TestRenderPriorWork` loses its failure-record rows.

`loomshed`'s `doc.go` drops the reroute and rejection paragraphs of Darn-Write.

**ImpactSummary:** Darn-Write's seam set shrinks to two; its only wiring is `darnDeps`, and no other row reads the removed seams.
