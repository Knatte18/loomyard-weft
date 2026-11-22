```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: for each change a card makes, I looked for every reader, test, doc and fixture of it and checked which cards cover them.

**Per change: what reads it, and which cards cover it**

- **Card 1** (`autonomy` key, `LookupAutonomy`, check branches): the template, ConfigSpec, `TestModules_EveryTemplateKeyClassified`, the tasksetting file comments, `doc.go`, the `storeVerbs` help and `PATTERN-config-layers.md` all sit on card 1. The only consumer is card 3. Covered.
- **Card 2** (`configChangeNotice` skips `beam.autonomy`): its one caller (`finalize.go:371`) and `TestFinalize_ConfigChangeNotice` are on card 2. Covered.
- **Card 3** (`hubgeom.Autonomy`): new symbol with no current readers. Its consumers are cards 4, 6, 15, 16 and 18. Covered.
- **Card 4** (halt notice bodies gain `; autonomy: &lt;v&gt;`):
  - Readers: the orch, which card 17's role text covers; `prepush_test.go:144-163` `wantNotices`; batten notice tests.
  - `prepush_test.go` injects its own `notify` seam (`prepush_test.go:280`), so the stamp added inside `newPrePushCheck`'s closure never reaches those assertions. Held.
  - Covered.
- **Card 5** (`ParentAsk.Cleared`): read only by card 6's gate. Tolerant JSON. Covered.
- **Card 6** (drift `Config.Autonomous`, the judge halt and the resume): the readers are the drift config readers `parentAgreement` and `JudgeTable`, both reached through driftDeps' `config` closure (`driftwiring.go:105,236-241`). Covered.
- **Card 7** (forced loom keys, new `pr_review_parent_timeout_min`): the template, spec, LoadConfig tests and the `docs/overview.md:1483` sentence are on card 7. Covered.
- **Card 8** (`Record.Autonomy`, `ClearReviewSession`): readers are cards 10, 11, 13 and 14. Covered.
- **Card 9** (deletes `parentReviewRequestPrefix`, exports the tag): references are `prreview.go`, `TestPRReview_Session` (card 9) and `contracts/orchrole/orch-role.md` (card 17). Covered.
- **Card 10** (new `RenderPRReviewPrompt` parameter, stencil rewrite, three new `PullRequestDeps` seams):
  - Callers `prsession.go:154` and `shuttleparity/producers_test.go:257` `buildPRReview` are both on card 10.
  - The seam waivers (both `intentionallyNil`), `envkit.LandingDeps` and `optionalPullRequestSeams` are on card 10.
  - No test asserts the stencil's text (search found none). Covered.
- **Card 11** (`gateRetryHint` wording, closed-PR branches, Finalize halt): the hint constant's only reader is `prreview.go:752`. `docs/overview.md:1385-1388` is on card 11. Covered.
- **Card 12** (`parent-review ask`, answer refusal text): `parentReviewFixture`, `TestParentReviewAnswerVerb` and `docs/overview.md` are on card 12. Covered.
- **Card 13** (`prContinueVerb` signature and behaviour): callers `prContinueCmd` and `TestPRContinueVerb` are on card 13. The file header of `prcontinue.go` is not (finding 2). Partly covered.
- **Card 14** (goto clears the record): the `GotoGuard` comment and `gotoGuardEndingFixers` are on card 14. Covered.
- **Card 15** (three new required Beam-Await seams and `Paths.DrainMarker`):
  - On card 15: `pawlDeps`, `envkit.FullEnv`, batten's `intentionallyNil`, `world_test` `world.deps`, `beamAwaitEntry`, `requirePawl`, `hubgeom.PawlPaths` and `TestDobbyGeometry`.
  - Missed: `internal/shedrecipe/fixture_test.go#validPawlDeps` (`fixture_test.go:67`) and the BeamAwait row of `entries_simple_test.go:239-247` (finding 1). Partly covered.
- **Card 16** (reply hook filter, new `slackCLI` field): `harness`, `newHarness`, `TestHook` and `wireProduction` are on card 16. Covered.
- **Card 17** (role, on-demand file, glossary, help texts): `TestRenderRoleFile_*` ranges over `OnDemand`, and `orch-role.md`'s `[pr-review request]` creel answer is on card 17. Covered.
- **Card 18** (`orchengine.Paths.Autonomy`, pointer text):
  - The six `WithPresence` sites (`start.go:836,854`, `resumecontext.go:43`, `watcher.go:1281,1351,1600`) are on card 18.
  - `orchroute.Paths` and `battencli/overview.go:516` build `Paths` but render no pointer. A nil field reads off. `orchcli/cli.go:254` fills `c.paths` through `orchPaths`, which card 18 edits.
  - Covered.

**Findings**

1. **BLOCKING**, class `consistency`, location `_lym/plan/15-beam-await-drained.md`.
   - Summary: card 15 makes `NewBeamAwait` and `beamAwaitEntry` require `Autonomous`, `HoldsCommits`, `WorkRemains` and `Paths.DrainMarker`. It misses shedrecipe's own fixture: `validPawlDeps` (`internal/shedrecipe/fixture_test.go:67-95`) fills none of them, and it feeds the BeamAwait happy path through `newTestEnv` (`fixture_test.go:315`).
   - Effect: the BeamAwait rows of `TestSimpleEntries_HappyPath` and the conformance table (`internal/shedrecipe/entries_simple_test.go:239-247`, `conformance_test.go:44`) fail once the seams are required. The row's `validatedFields` list also omits the new seams.
   - Evidence: `internal/shedrecipe/fixture_test.go:67-95`, `internal/shedrecipe/entries_simple_test.go:239-247`.
   - Fix: add `internal/shedrecipe#validPawlDeps` and `internal/shedrecipe#simpleEntryCases` (or whichever function holds the table at `entries_simple_test.go:239`) under card 15's Edit. Fill the three seams and the drain marker path, and add the four field names to the BeamAwait row's `validatedFields`.
2. **LOW**, class `consistency`, location `_lym/plan/13-pr-continue-autonomy.md`.
   - Summary: card 13 changes what `pr-continue` means under autonomy and updates its Long help and `doc.go`. The file header of `internal/loomcli/prcontinue.go:1-7` stays false: it says the verb tells "the run the operator is satisfied", and card 13 does not list that file as a target.
   - Evidence: `internal/loomcli/prcontinue.go:1-7`.
   - Fix: add `internal/loomcli/prcontinue.go`'s file header to card 13's Edit list, or have the Intent name the header rewrite.

**Tool calls**

1. Grepped `RenderPRReviewPrompt` callers, `[pr-review request]` readers, the gate-hint text, `pawlengine.Deps` and `orchengine.Paths` literals, and the PR-Review doc lines: the manifest had none of these readers.
2. Read the shedrecipe `validPawlDeps` fixture and its BeamAwait tests, the watcher's paths source and `buildPRReview`'s location, and grepped for stencil-text tests: these are readers outside the manifest.
3. Read the BeamAwait row of `entries_simple_test.go`, `validPawlDeps` usage, and the notice-body tests: needed to confirm finding 1 and card 4's claim that no test asserts these bodies.
4. Read `prepush_test.go`'s notice assertions: needed to settle whether card 4's stamp reaches them.
