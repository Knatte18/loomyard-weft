# Card 12 — darn-pr-align-wiring

**Edit:**
- `internal/loomcli#landingAlignmentDeps` -> `func landingAlignmentDeps(l *lyxcwd.Location, recipe, stencilsDir string, registry modelspec.Registry, runner *shuttleengine.Runner, pause pauserecord.Geometry) landingshed.AlignmentDeps`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, recipe string, geom websterengine.Geometry, taskBranch, originURL, parentBranch string, pushSkipped bool, pushBranch func() error, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error), verify verifySource, lock landingshed.ParentLockDeps) landingshed.Deps`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/loomcli#markDoneFixtureDir`
- `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/loomcli#TestLandingAlignmentDeps_Config`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/alignment#MeasureLoom`
- `plan:internal/alignment#MeasureDarn`
- `internal/loomengine#DiscussionDecisionRecord`
- `internal/loomengine#DiscussionSupportLog`
- `internal/discussionparser#AppendDecision`

**Intent:** Wire PR-Align with the run's own geometry, so a darn run's alignment agent gets the darn record and support log, `none` for the plan directory and darn's measure (D9), while reading loom.yaml's alignment keys as a loom run does.

`landingAlignmentDeps` takes the run's recipe.
For `shedrun.RecipeDarn` its `Prompt` carries an empty `PlanDir` and `MeasureStencil: alignment.MeasureDarn`; for every other recipe it keeps `planparser.PlanDir(l.AnchorPath())` and `alignment.MeasureLoom`.
`DecisionRecordPath`, `SupportLogPath`, the snapshot over the decision record and the board notes, the session root, the note path and the `Config` closure, which loads loom.yaml's `skip_alignment`, `alignment`, `alignment_timeout_min` and `alignment_pr_approve` on each call, are the same for both recipes (A2): an agent approve on a darn run lands it under `alignment_pr_approve`, `skip_alignment`, the snapshot and head checks and one agent reject per run, as `landingshed.PRAlign` already bounds a loom run.
`landingDeps` takes the recipe after the location and passes it to `landingAlignmentDeps`; `loomCLI.loomPreRun` and `loomCLI.buildLoomShed`, the Go driver's shed build, both pass `c.recipe`; every test caller passes `shedrun.RecipeLoom`.

`TestLandingAlignmentDeps_Config` takes the recipe in its `configOf` helper and runs its `skip_alignment` and `alignment_pr_approve: false` rows for both recipes, pinning that the loom.yaml keys leave a darn PR-Gate to the parent as they do a loom one.
It gains a subtest over both recipes on one location asserting each prompt's `DecisionRecordPath` and `SupportLogPath` equal the loomengine accessors' paths, the darn prompt's `PlanDir` is empty and its `MeasureStencil` is `MeasureDarn`, and the loom prompt's are the plan directory and `MeasureLoom`; the rendering of an empty plan directory as `none` is `internal/alignment#TestPrompt`'s darn row (card 11).
The same subtest writes a decision record at the darn location, reads the darn deps' `Snapshot`, appends an `Added after Discussion` entry through `discussionparser.AppendDecision`, and asserts the snapshot changed, so a `decision add` recorded while the darn agent runs turns its approve into an escalation through `PRAlign`'s existing snapshot check (`TestPRAlign`'s "an approve over a changed snapshot leaves a note and no record" row).
An agent reject on a darn run needs no new wiring: `landingDeps` tells PR-Align `RejectionPath: loomengine.LoomRejectionPath(l)` for both recipes, the file `pendingRejectionReader` (`internal/loomcli/wiring.go:304`), the darn writer's `ReadRejection` seam, reads.
`internal/loomcli/doc.go` gains a "PR-Gate alignment" paragraph beside "Parent-review alignment": PR-Align's session, its snapshot and the loom.yaml keys it reads, and that a darn run's prompt names no plan directory and carries darn's measure.

**ImpactSummary:** the two landing-deps assemblers take the recipe; a loom run's deps are unchanged, and every caller is the pre-run or a test passing loom.

**Commit:** `12: PR-Align is wired with darn's geometry on a darn run`
