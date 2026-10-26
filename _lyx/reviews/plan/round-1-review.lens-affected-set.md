```yaml
lens: affected-set
outcome: reported
```

Task: review the plan through the affected-set lens, finding what each card's change forces outside the card and which cards cover it.

**Per change: the set found and the cards that cover it**
- Removed-verb strings in production code (`lyx orch start`, `orch cleanup`, `lyx prime`, `beam promote`, `adopt --prime`):
  - The production files are covered by cards 1, 4, 7, 9, 10, 11, 12, 25 and 26. That includes `beamcli/cli.go:151` (startCmd, card 11), `adoptcli/cli.go:52` (Command, card 4), `cleanupcli/cli.go:42` (card 9), `battencli/cli.go:20/503/519` (cards 26 and 7), `fabricengine/destroy.go:2400` (inside the deleted `ResetRecordsToParent`) and `loomcli/beamwiring.go:57` (card 1).
  - The asserting tests are covered by cards 1, 7, 10, 12, 25 and 26: TestExitCode_HelpPaths, TestShortcuts_FileAndRootListing, TestAdopt, TestCancelRun, TestStartFirstPass, TestStartGuard_Refuse, TestCreate, TestLand, TestSetOwners, TestUpsertFieldAllowlist, TestEnsureLive, TestChangeAnchor, TestReconcileOrigin_Moves, TestWatch, TestFinalize_OwnerChecks, TestRefuseNonPrime, TestStatus_RestartsADeadWatcher, the TestStart_* family, TestWatcherIndexSource, TestSmokeOrch_*, liveFixture.startOrch and the shortcutcli tests.
  - Not covered: test-file comments only (finding 4).
- `shedrun.RecipeBeam`, `NewBeam`, `BeamRouting`, `BeamRecipe` and `beamTacklerRecipe` removed (card 6):
  - Production: loomcli `arm.go:198/228/245`, `wiring.go:827`, `drivewiring.go:99/120`, `landingdeps.go:78/249/264/293/303`, `arbiterprompt.go:22`; loomrecipe `names.go`; battencli `wire.go:497-513`; `contracts/recipes/recipes.go:24`.
  - Tests: TestRecipeRouting, TestRowNames_FollowEachRecipesRowOrder, TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree, TestBoardRecipe, TestValidateRecipe, TestRecipeNames, TestNewBeam, TestBeamRouting, TestWire_BeamRecipe.
  - All on card 6.
- Removal of `SourceFlag`, `SourceAdopt`, `RenderAdoptPrompt` and `OrchTemplateAdopt` (card 10): readers are `orchengine/launch.go`, `prompt.go:130`, `contracts/stencils/stencils.go:267,491`, `prompt_test.go:198/227`, `launch_test.go:58`, `start_test.go:758` and 1344-1444, and `smoke_live_test.go:480-502`. All are on card 10.
- `dobbyengine.EnsureLive` re-sign (card 7): callers `adoptcli/deps.go:122`, `fabriccli/anchordeps.go:272`, `beamcli/cli.go:160`, orchcli `ensureDobbies`, and `start_test.go` TestEnsureLive. All on card 7.
- `battencli.StartFirstPass` re-sign (card 18): callers `adoptcli/deps.go:95`, `beamcli/deps.go:143` and TestStartFirstPass. All on card 18.
- Card 3 dropping task owners on load, on normalize and in `ownersOf`. Test fixtures that give a task its own owner:
  - Covered by card 3: TestOwnersEffective, TestRenderReadmeGolden, TestListTasksBriefLayerAndProposal, TestNewTask, TestSetOwners, `ownedBy`, `fakeOwners.seams`, TestArmAt_RunOwnedElsewhereWritesNoSeed, TestPassExitCode and TestPairOwners.
  - Covered elsewhere: TestAdopt (card 4), TestCreate (cards 1 and 21), TestTeardown_OriginSeams (card 25). TestBattenStatus_CarriesTheOwner goes through `ownedBy`, so card 3 covers it.
  - Missed: TestFabricReplay, TestRunLock's owner row, and boardcli `stepOwners` (finding 1).
- Card 25's owner way forward moving from `lyx adopt <slug>` to `lyx adopt <subject>`:
  - Listed tests: TestReconcileOrigin_Moves, TestRunCmd_ResumeDispositions, TestBattenPreStep_RefusalKinds, TestWire_CreateWorktreeClaimsTheBoardEntry, TestCancelRun, TestInnerRun_OwnerCheck, TestOwnedElsewhere, TestWatch, TestStartAndResume_RefuseARunOwnedElsewhere, TestFinalize_OwnerChecks, TestChangeAnchor.
  - Missed: TestRefuseStartOwnedElsewhere (finding 2).
- Card 1 deleting `ResetRecordsToParent`: readers are `fabricengine/doc.go` (card 1), `beamcli/deps.go:109` and `beamland_integration_test.go` (card 1), and the `destroy.go:2` file header, which no card targets (finding 3).

**Findings**

1. MEDIUM, design — card 3 (owners-on-projects). Card 3 covers only part of the set of test fixtures that rely on a task carrying its own `owner_machine`. Three members break at card 3's gate:
   - `internal/boardengine#TestFabricReplay` (`internal/boardengine/replay_test.go:339-356`) asserts that replaying entry "a" keeps `owner_machine` and reports a lost `owner_machine` field. Card 3's `normalizeTask` and `migrateEntries` clear that key on any entry that is not a beam project.
   - `internal/boardengine_test#TestRunLock`'s row "upsert naming the held entry's own effective owner" (`internal/boardengine/runlock_test.go:103`) gives the held task owner "box" while the prime owner is "neve". Once the task owner is dropped, its effective owner reads "neve", so `checkOwnerPayload` refuses the "box" upsert. Card 5 edits TestRunLock, but card 3 lands first.
   - `internal/boardcli#stepOwners` (`internal/boardcli/verbs_test.go:366-383`) seeds a task `"owned"` with `owner_machine:"box"` and asserts `get` and `list` report "box". After card 3 they report the prime's owner.

   Fix: add these three glyphs to card 3's Edit list, moving each task owner onto a beam project or the prime.
2. MEDIUM, design — card 25 (adopt-subject-ways-forward). `internal/battencli#TestRefuseStartOwnedElsewhere` (`internal/battencli/owner_test.go:47,61,101`) asserts `"lyx adopt " + slug`. Card 25 rewords `refuseStartOwnedElsewhere` to name `lyx adopt <AdoptSubject>`. Over card 3's `ownedBy` fixture the subject is `some-beam`, not the slug. The test is not on card 25's list.

   Fix: add `internal/battencli#TestRefuseStartOwnedElsewhere` to card 25's Edit list and assert the beam subject.
3. NIT, consistency — card 1 (remove-beam-promote). The file header at `internal/fabricengine/destroy.go:2` lists "replacing a pair's records content with its parent's (ResetRecordsToParent: …)". Card 1 deletes that function but edits only `fabricengine/doc.go`, so the header goes stale. A file-level target beside card 1's member glyphs of `destroy.go` may be refused by `redundant-file-target`.
4. NIT, consistency — cards 9 and 10. Several test-file comments still name the removed verbs:
   - `internal/orchcli/startdobbies_test.go:3` file header ("pins that `lyx orch start` starts the dobbies").
   - `internal/orchcli/pausegeometry_test.go:17` ("`lyx orch start` types its start lines").
   - `internal/cleanupcli/cli_test.go:92` ("as `lyx orch cleanup` does").

   No card targets them, and card 26's scan reads production files only, so they stay stale silently.

**Tool calls**
1. A grep of the removed-verb strings in test files, with each hit's enclosing function. The pre-read covered production files only.
2. A grep of the uncovered test hits and of every caller of `RecipeBeam`, `NewBeam`, `BeamRouting`, `SourceFlag`, `SourceAdopt`, `RenderAdoptPrompt`, `OrchTemplateAdopt`, `EnsureLive` and `StartFirstPass`. The pre-read lacked the caller sets.
3. The enclosing tests of eight hits from call 2, and the list of test files with owner fixtures, to map hits to card glyphs.
4. The enclosing functions of every task-owner fixture in tests, to check card 3's fixture coverage.
5. The owner and adopt lines in owner_test, verbs_test, replay_test, runlock_test and store_test, to confirm what the uncovered fixtures assert.
6. The enclosing test of `owner_test.go:47/101`, the fixture of TestBattenStatus_CarriesTheOwner, and the test holding `replay_test.go:330`, to pin the missed glyphs.
