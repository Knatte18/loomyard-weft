# Card 1 — remove-beam-promote

**Delete:**
- `internal/beamcli/promote.go`
- `internal/beamcli/promote_test.go`
- `internal/beamcli#promoteCmd`
- `internal/beamcli#hubPromoteDeps`
- `internal/battencli/promote.go`
- `internal/battencli/promote_test.go`
- `internal/battencli#runPhases`
- `internal/fabricengine#ResetRecordsToParent`
- `internal/fabricengine#ResetRecordsResult`
- `internal/fabricengine#resetRecordsSubjectPrefix`
- `internal/fabricengine#ParentRecordsAnchor`
- `internal/fabricengine#rootPairNoParentError`
- `internal/fabricengine/resetrecords_integration_test.go`
- `internal/fabricengine/parentrecordsanchor_integration_test.go`
- `internal/boardengine#turnIntoProject`

**Edit:**
- `internal/beamcli#Command`
- `internal/beamcli#createCmd`
- `internal/beamcli#createSteps`
- `internal/beamcli#judgeCreateEntry`
- `internal/beamcli#judgeLandEntry`
- `internal/beamcli#createWorld.deps`
- `internal/beamcli#TestCreate`
- `internal/beamcli#TestLand`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestBeamLanding_ReachesWorktreeCreateSkip`
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#checkBeamPayload`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine_test#TestUpsertFieldAllowlist`
- `internal/orchcli#refuseNonPrime`
- `internal/orchcli#TestRefuseNonPrime`
- `internal/loomcli#beamCommitSubjects`
- `internal/boardcli#Command`
- `internal/beamcli/doc.go`
- `internal/boardengine/doc.go`
- `internal/fabricengine/doc.go`
- `plugins/ly/skills/board/SKILL.md`
- `docs/overview.md`

**Intent:**
D13: `lyx beam promote` goes, with every helper only it calls and every way forward naming it, since a beam is made from its project, never from a running task.

- `beamcli`: the `promote` verb (`promoteCmd`, `hubPromoteDeps`, the whole of `promote.go` and its test) is removed and `Command` no longer adds it.
  `orchWayForward` lived in `promote.go`; `createSteps` writes its way forward text inline, unchanged in wording (card 10 rewords it and card 21 rewrites create).
  `judgeCreateEntry`'s task refusal and `judgeLandEntry`'s task refusal stop naming `lyx beam promote` and name a project entry instead: make one with `lyx board upsert` as a project under a slug of its own, then `lyx beam create <that slug>`; `createCmd`'s help drops its promote sentence.
  The `createWorld.deps` fake stops reading `OwnerRequest.ToProject`, and `TestCreate`'s `TaskRefused` row and `TestLand`'s `TaskRefusedNamingItsOwnLanding` row assert the new wording and that no `lyx beam promote` appears.
- `battencli`: `EndRunForPromote`, `ClearRunForPromote`, `ErrPromoteEndOnly`, `promoteRerun` and their tests go with `promote.go`; `TestCancelRun` drops its three rows that pass a promote re-run and the `runPhases` helper only they call.
  The start guard keeps refusing a run or step of a task whose local pair carries the beam mark, which only an interrupted promote of an older build leaves, and its way forward becomes `lyx batten cancel <slug> --remove` run from the prime, then the release and the run again (D11); `TestStartGuard_Refuse`'s two rows asserting `lyx beam promote some-slug` assert the cancel instead.
  `TestBeamLanding_ReachesWorktreeCreateSkip` drops its `prom` pair, the `MarkBeam` and `ResetRecordsToParent` setup and the `ToProject` owner request, and keeps its other beams and cases.
- `fabricengine`: `ResetRecordsToParent`, `ResetRecordsResult`, `resetRecordsSubjectPrefix` and `ParentRecordsAnchor` have no caller left but promote, so they go with their integration tests; `rootPairNoParentError` is called only by those two and goes too, while `missingOriginRecordError` stays for `MarkBeam`.
  `fabricengine/doc.go`'s records-reset paragraph is removed.
- `boardengine`: `OwnerRequest.ToProject`, its refusal in `SetOwners` naming `lyx beam promote`, `turnIntoProject` and `writeOwners`' handle assignment for it go; `checkBeamPayload`'s refusal of a beam key on a task names making a project entry and `lyx beam create <slug>` instead of `lyx beam promote`.
  `TestSetOwners` drops its `ToProjectTurnsATaskIntoABeamProject` and `ToProjectWithoutBeamIsRefused` subtests, and `TestUpsertFieldAllowlist`'s two beam-on-a-task rows assert the new wording.
- `orchcli.refuseNonPrime` names only `lyx beam create <slug>` to make a project a beam, and `TestRefuseNonPrime` asserts no `lyx beam promote`.
- `loomcli.beamCommitSubjects` names only `lyx beam create` in its way forward.
- `boardcli.Command`'s long help says the beam flag is set only by `lyx beam create`.
- Docs: `beamcli/doc.go` loses its promote section and every mention of promote; `boardengine/doc.go` loses the `ToProject` sentence and names only `lyx beam create`; the board skill says the `beam` field is set only by `lyx beam create`; `docs/overview.md`'s beam module line drops `lyx beam promote` and the promote sentence.

The guard change bounds nothing new: the start guard still refuses the same marked-pair task, only its way forward changes.

**ImpactSummary:** removes one verb and its private helpers across beamcli, battencli, fabricengine and boardengine; the only external callers were promote's own deps, and every reworded refusal has its asserting row on this card.

**Commit:** 1: Remove `lyx beam promote` and the helpers only it called
