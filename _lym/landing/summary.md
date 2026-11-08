# Watch patterns on guard tests so a card gate skips unaffected guards

The guard step is most of the card gate's time.
`docs/research/webster/webster-time-2026-11-08.md` measured it at 7,043 s of the card gate's 9,117 s over 14 Webster runs:
787 guard runs for 226 cards, about 8.4 min per run.
The card gate keeps the guard step, so a guard failure still names the card that caused it.
This change cuts the time by running fewer guard tests per card.

A `//lym:guard` test marked `//lym:watch <pattern>...` is left out of a narrowed gate
when no changed file matches a pattern and the test's own file did not change.
Few guards carried the marker.
This change adds it to every guard whose outcome only a bounded set of files can change.
A whole gate (plan-level verify, whole scope) still runs every watched test, so no guard leaves the gate entirely.

Each newly marked guard also gains a doc-comment line saying when a narrowed gate runs it.
`pattern/PATTERN-test-speed.md` records the rule for marking a guard and two bounds that follow from it.

## Design decisions

- **Patterns cover every file the test reads.**
  That means the files it scans, a package declaring a value it names, and a helper in another file.
  Every scan-based guard watches `internal/testkit/scankit/*.go`, because a change to the shared scan helpers can change its outcome.
- **Recursive walks reach one directory level below the deepest file they scan today.**
  `path.Match` has no `**`, so the patterns are written per depth, for example `internal/fabricengine/*.go internal/fabricengine/*/*.go`.
- **Unbounded guards stay unmarked.**
  A guard that scans all Go code of the module, or of `internal/` and `cmd/`, or whose inputs come from the scanned text, has no pattern set that bounds it.
- **Bound: depth.**
  A file nested deeper than a guard's patterns reach does not run it in a narrowed gate; a whole gate does.
- **Bound: test name.**
  The skip matches by test name, so a test of the same name in another package of the run is skipped too (existing behaviour, now noted in the pattern doc).

## Guards that got patterns

| Test | Patterns watch |
|---|---|
| `TestBoardGuard_NoRawGitImportOrShellOut` | `internal/boardengine` to two levels below, `scankit` |
| `TestCwdMutation_MigratedFilesStayChdirFree` | the subject-set test files it names, `scankit` |
| `TestCwdMutationGuard_NotVacuous` | the one allowlisted file, `scankit` |
| `TestNoDestructiveBypass_FabricengineProductionSource` | `internal/fabricengine` to one level below, `scankit` |
| `TestMutationRecord_FabricengineProductionSource` | `internal/fabricengine/*.go`, `scankit` |
| `TestGitrepoBoundary_PinnedRunCallSites` | `internal/gitrepo/*.go`, `scankit` |
| `TestNoDeniedRecovery_ScannedSurfaceHasNone` | specs, stencils, `plugins/ly/skills`, `internal/webster*`, `internal/shed*`, `internal/loom*`, `internal/shellguard`, `scankit` |
| `TestWebsterRetiredSteps_ScannedSurfaceHasNone` | specs, webster stencils, `internal/webster*`, `norecoverydeny_test.go` (shared token matcher), `scankit` |
| `TestNoRawGitMutation_WebsterProductionSource` | `internal/websterengine` to one level below, `scankit` |
| `TestRemoteRefsLock_FabricengineFetchesWritingTrackingRefsGoThroughGitrepo` | `internal/fabricengine/*.go`, `scankit` |
| `TestSandboxCoverage_AllModulesCoveredOrExcluded` | `tools/sandbox/*SUITE.md`, `cmd/lym/main.go`, `internal/*cli/*.go`, `internal/shortcutcli/*`, `scankit` |
| `TestNoUncontainedWrite_FabricengineProductionSource` | `internal/fabricengine` to one level below, `scankit` |
| `TestVerifiedTree_PlanVerifySitesCallVerify` | the four plan-verify site files, `scankit` |
| `TestBannedDeclarations_CliPackagesCallIntoCliwire` | `internal/webstercli`, `internal/burlercli`, `scankit` |
| `TestRefKindScan_RealTreeClean`, `TestPlanOpScan_RealTreeClean` | `internal/planparser`, `internal/planglyph`, `scankit` |
| `TestEnforcement_ScankitStdlibOnly` | `internal/testkit/scankit/*.go` |
| `TestStencils_NameGoValues` | stencils, `contracts/orchrole`, the stencil registry, and the packages declaring the values it names |
| `TestTwillStencils_HostNeutral`, `TestHubGenericStencils_NameNoGoTiers` | stencils, `contracts/orchrole`, the stencil registry |
| `TestStencils_OpeningStencilsRenderParentDirective`, `TestStencils_OpeningStencilsRenderEditDirective` | the stencil set above, plus `internal/parentdirective`, `internal/editdirective` |
| `TestStencils_NoEditRuleRestatement`, `TestStencils_NoUnboundedAskOperator`, `TestStencils_NoSkillLoadInstructions` | stencils, `contracts/orchrole`, the stencil registry |

`TestCSharpFixtures_CompileWithDotnet` already watched `*.cs`.

## Guards left unmarked

Each of these scans all Go code, or all of `internal/` and `cmd/`, or reads inputs no pattern set bounds, so a change anywhere can fail it:

- `TestRegistration_MatchesDeclarers`, `TestRegistration_AllModulesRegistered`, `TestConfigStrictness_PinnedCallSiteSets`
- `TestProviderSeamImportRule`, `TestSeamMarkers_ReflectiveTestsAreMarked`, `TestCheckedCallInvariant_RawSitesMarkedAndPinned`
- `TestEnforcement_PairTeardownChokepoint`, `TestDriverChoiceSingleSiteInvariant_OnlyLoomcliReadsTheSeedDriverField`
- `TestCopyRepoCallerSet_LymcwdOnly`, `TestDeriveCallerSet_CliwireOnly`
- `TestEnforcement`, `TestEnforcement_GeometryLiterals`, `TestEnforcement_BoltDirNameDeclaredOnce`, `TestEnforcement_FabricVocabulary`, `TestEnforcement_LocationLiterals`, `TestEnforcement_MarkdownLinks` (link targets come from the link text)
- `TestEnforcement_TestkitInvariant`
- `TestGofmt_ModuleFilesAreFormatted`, `TestDaemonWakeups_UnboundedWaitsStayAboveTheFloor`, `TestFslinkOnly_NoRawLinkCalls`
- `TestSpawnObservability_ProductionSpawnsAreLogged`, `TestAgentFiledIssues_NoCreateIssueOutsideSelfreport`, `TestSpawnRole_NoInlineRoleLiteral`
- `TestTierTags_OneTierPerTestFile`, `TestTierPurity_UntaggedTestsSpawnNothing`, `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain`
- `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain`, `TestLLMTier_OnlyLLMFilesReachAnLLM`, `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt`
- `TestGlyphConversionChokepoint_NoLocalConversion`, `TestGHGuard_NoShellOutOutsideGithubclient`, `TestPullRequestGuard_OnlyLandingAndTeardown`
- `TestTolerantRecords_NoStrictJSONDecode`, `TestNoCgo_NoFileOfTheModuleImportsC`, `TestShuttleStop_RemoveStrandOnlyInAllowedPackages`
- `TestInteractiveSpawn_NoHeadlessClaude`, `TestBareTemp_NoCreationInTheBareSystemTempDirectory`
- `TestVerifiedTree_NoOtherVerifyRunCaller`, `TestVerifiedTree_NoRetiredMarker`

## To check by hand

- **Skip estimate not computed.**
  The decision asked for an estimate of the guard runs a card gate would have skipped over the 14 runs.
  The changed files of those runs live in the runs' archive tags, which this change did not read, so no count is given.
  The report's own figures bound it: the guard step ran 787 times in card gates.
  Compute the skip count by replaying each run's card diffs against the new patterns.
- **Pattern coverage.**
  Spot-check that each marked guard's patterns cover every file it reads, since a missed file means a card that breaks the guard passes its card gate.
  The whole gates still catch it, but later and without naming the card.
  The scan-based guards with the widest patterns (`TestNoDeniedRecovery_ScannedSurfaceHasNone`, `TestWebsterRetiredSteps_ScannedSurfaceHasNone`, `TestSandboxCoverage_AllModulesCoveredOrExcluded`) are the likeliest to have a gap.
