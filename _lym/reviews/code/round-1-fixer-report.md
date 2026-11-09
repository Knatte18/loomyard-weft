```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6, F8, F9]
disputed: [F7]
deferred: []
```

## Changes

Commits group findings by package, since several findings change the same file.

- **F1** (`boardengine`, add421c1): `TestFileRaddle` gained the row `TasksShapeMovesOpenDependentsOntoTheSinkPartAndLeavesDoneOnes`, with a `setup` field on the table.
  An open `waiter` depending on `other` and `plan` ends depending on `other` and `plan-b`; a done `finished` stays on `plan`.
- **F2** (`loomshed`, 2354f000): `Call` clears `entry.StatusProducer` after `repairFamily` succeeds, so `stageAndFile` never claims a status back onto the converted project.
  The repair table gained the row `WithTheStoreWriteLostAndAnotherProducersStatusOnTheEntry`, whose steps hold no `claim_status`.
- **F3** (`boardengine`, add421c1): `endsAsFamilyProject` also requires `now` to have no profile, no run settings and no notify-on-land.
  `TestCheckRunLocks_FamilyProject` gained a refusal row for each of the three.
- **F4** (`loomcli`, 4262d16c): `TestCommitStatusDeps_SkipsTheBoardStatusAfterAFilingMark` builds `commitStatusDeps` over a real hub.
  Its rows: an arras run without the mark writes; with the mark it writes nothing; a tapestry run writes despite the mark.
  I ran it at tier 1 and it passed.
- **F5** (`boardengine`, add421c1): the family subtest now writes a real run status before each null status write.
  The writes are `SetStatus`, `UpsertTask` and `MergeTasks`; none releases a member.
  The batten cancel claim clear has no board method of its own, so `SetStatus` with a nil status stands for it.
- **F6** (`boardengine`, `loomshed`):
  - Lock rows: a changed order, a changed body, a released part, a part already loaded and no added part.
  - A `repointDependents` subtest for the `validateWrite` refusal, built over a store whose sink the board does not hold.
  - `ClearProjectRunStatus` no-op checks for an absent entry, a task and a project with no status.
  - The repoint-refusal subtest is now a table that includes an entry with no former record.
  - A `loomshed` subtest where a failing `mark_filed` or `clear_status` stops the repair.
- **F8** (`docs/overview.md:1302`, `wiring.go:232`, `loomrecipe/doc.go:55`): added the family filing exception.
- **F9** (`contracts/recipes/raddle-recipe.yaml:4`, `boardengine/raddlefile.go:2`): reworded the same way.

A later gate flagged two comment lines of 120 characters or more, in `commitstatus_integration_test.go` and `loomrecipe/doc.go`.
A second gate flagged the first split as a wrapped break mid-sentence.
Both comments are now rewritten as whole sentences, one per line, each under 120 characters; no behaviour changed.

The gate passed for `boardengine`, `loomshed` and `loomrecipe` at tier 0, and for `loomcli` at tiers 0 and 1.

## Disputed

- **F7**, ground `premise false`: `Call` returns `p.fail("parse the proposal", parseErr)` at `internal/loomshed/raddlefile.go:205-207` whenever `parseErr != nil`.
  That check comes before `finish`, so a done store with an unparsable proposal never reaches `EndEntry`.
  The fix key's early return would duplicate it.

## Deferred

- `internal/fabricengine/replay.go:250` and `internal/fabricengine/doc.go:1761` (raised by the consistency, proposal and affected-set lenses) state that Raddle-File ends the run's own entry done, and describe reconcile logic that reads an entry shown done at origin.
  Whether that reconcile handles a family run, whose entry is a project that never reaches done, is a behaviour question the review did not raise.
  Rewording the comments alone would hide it. It is left for the parent or the operator to settle.

## Rulings

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, twins of the entry-ends claim in `loomrecipe/doc.go:55`: fixed with a family exception.
- fix-consistency, LOW, `replay.go:250` and `fabricengine/doc.go:1761`: deferred, as above.
- fix-consistency, LOW, `runlock.go` `checkRunLocks` doc not naming the three cleared fields: no change; `endsAsFamilyProject`'s own doc names them and the lock doc defers to it.
- fix-consistency, no finding on `loomshed/raddlefile.go:139` and `boardengine/raddlefile.go:2`; the header of the latter was reworded anyway on the affected-set lens's report.
- fix-proposal, LOW, `replay.go:250` and `fabricengine/doc.go:1761`: duplicate of the above, deferred.
- fix-affected-set, LOW, `loomrecipe/doc.go:55` and `boardengine/raddlefile.go:2`: fixed.
- fix-affected-set, LOW, `replay.go:250` and `fabricengine/doc.go:1761`: duplicate, deferred.
