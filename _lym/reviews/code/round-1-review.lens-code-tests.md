```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke. I made one tool call, a `git diff` of the test files.

**Per behaviour, the test that pins it**

- **Body lock, `ProjectRecordHeld` and `checkRunLocks`:** `TestRunLock` in `runlock_test.go` pins it. The rows cover Preflight (editable), Record-Write, Beam-Await and blocked Beam-Await (refused), and the error type and message. Title, brief, labels and membership edits stay editable in the existing Beam-Await rows.
- **Record-free and hold-free rows against the recipe's row names:** `TestPawlRecipe_BodyLocksFromRecordWrite` and `TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign` in `pawlrecipe_test.go` pin it.
- **Pawl recipe rows and edges, no review segment, `Routing` and `New` without a Code-Review budget:** `TestPawlRecipe` in `pawlrecipe_test.go` and the `TestWire_BeamRecipe` rows list in `wiring_beam_test.go` pin it.
- **`BeamAwait` as a build row, removal of `NewPawl`, `PawlRouting` and `ServedByLoom`:** `TestPawlRecipe` covers `New` accepting pawl. I could not see the `conformance_test.go` or `shedbuild` test diffs, so the `onlyRowOf` message change is unconfirmed.
- **Escalate judged first for pawl:** the `wiring_beam_test.go` landing-deps row flipped to `true` pins it.
- **Record paths (`ProjectDir`, `ProjectDecisionRecord`, `ProjectSupportLog` and the Rel forms):** `TestDiscussionPaths` pins them.
- **`runRecordOf` choice by recipe:** `TestRunRecordOf` in `projectrecord_test.go` pins it for pawl, darn and tapestry.
- **`projectRecordPaths` (chain nearest first, skips a beam with no record, none under no beam):** `TestProjectRecordPaths` pins it.
- **`ProjectRecordDirective` (empty for no records, bullets in order, banner stripped, missing stencil errors):** `TestProjectRecordDirective` pins it.
- **Directive stencil marker set:** `markers_test.go` pins it.
- **Beam-Describe `## Project decisions` section and the `ProjectDecisions` seam error:** `TestBeamDescribe` in `beamdescribe_test.go` pins both.
- **Decision `recordUnderLym` threading:** only the fake in `decision_test.go` was updated. See F1.
- **`decisionParent`, `decisionReceivers` and `memberPairs`:** none. See F1.
- **`withProjectDirective` (fills the marker for `DiscussionTable` and `DarnTable`, byte-identical prompt under no beam):** none. See F2.
- **`commitRunRecord` and `recordHistory` on a project record:** none. See F3.
- **`runRecordOf` consumed at wiring sites (`wireLightweight` `DecisionRecordPath` and `PatternRecordPath` empty for a beam, `recordWriteDeps` path, arbiter `Deps` record, `landingDeps` `DecisionRecordPath`, `beamDescribeDeps` record path):** none seen. See F4.
- **Darn, `pawlengine` and `envkit` snapshot removal:** deletion only, nothing to pin.

**Findings**

- **F1 [MEDIUM, scope]** `internal/loomcli/decision.go` (`decisionParent`, `decisionReceivers`, `memberPairs`, `preRun` record resolution) has no test. The diff only adds `recordUnderLym` to the fake in `decision_test.go:~150`. The behaviours with no test row:
  - the beam's own orch is the allowed caller;
  - the notice reaches the live strands of the beam's member pairs, which is the headline requirement;
  - the restore and differs way-forwards name the project record's real path (`discussion/...` is the only value any test uses).
- **F2 [MEDIUM, scope]** `internal/loomcli/projectrecord.go` `withProjectDirective` has no test:
  - nothing checks that `project_directive` is filled for a member under a beam and empty under none;
  - nothing checks that the marker is accepted as optional by `DiscussionTable` and `DarnTable` (the `Optional` list change in `darnengine/spec.go:112` is unasserted);
  - nothing checks that the prompt is byte-identical under no beam.
- **F3 [LOW, scope]** `commitRunRecord` (the project branch) and `recordHistory` in `projectrecord.go` are untested. A wrong pathspec or commit message would pass every test.
- **F4 [LOW, scope]** The project-vs-discussion record choice is untested at the wiring sites in `wiring.go`:
  - `wireLightweight` `DecisionRecordPath`, and `PatternRecordPath` staying empty for a beam;
  - `recordWriteDeps` paths for pawl;
  - `arbiterDepsFor` (`arbiter.go:~151`);
  - `landingDeps` `DecisionRecordPath` (`landingdeps.go:255`);
  - `beamDescribeDeps` record path.
  The pawl row in `wiring_beam_test.go` asserts only the escalate flags. A site that still read `_lym/discussion` for a beam would pass.
- **F5 [LOW, consistency]** Test economy: `internal/loomengine/projectdirective_test.go`, in the `RecordsRenderAsBulletsInOrder` subtest, loops over `"## Project record"` and `"lym loom decision add &lt;beam-slug&gt;"`. Those are new assertions on a stencil's static prose, which the rubric names as a finding. Fix: drop the `wantText` loop and keep the bullet order and the banner and marker checks.
- **F6 [NIT, consistency]** In `internal/loomengine/discussionpath_test.go`, the `ProjectDecisionRecord(...)` and `ProjectDir(...)` asserts against `ProjectDecisionRecordRel` and `ProjectDirRel` repeat the asserts just above them against `wantProject`, with no new behaviour. The two-beams-differ check is the only new coverage there.

Tool calls:
1. `git diff` of the new and changed test files, because the inherited context held no test diffs.
