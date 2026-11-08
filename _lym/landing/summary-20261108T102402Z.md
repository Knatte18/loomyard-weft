# Watch patterns on the guard tests whose scans read a bounded file set

A card gate's guard step runs every `//lym:guard` test by name, whatever the card changed.
A guard test that also carries `//lym:watch <pattern>...` is left out when no changed file matches a pattern and its own file did not change.
This change gives a watch marker to the 25 guard tests whose outcome only a bounded set of files can change.
The other 40 guard tests stay unmarked, because they scan all the Go code of the module, or of `internal/` and `cmd/`, or read inputs no pattern set bounds.
Whole gates (plan-level verify, whole scope) run every watched test, as before.

The patterns cover every file that can change a test's outcome:
the files it scans, a package declaring a Go value it names, a helper in another test file, and `internal/testkit/scankit` for the tests that walk through it.
A recursive walk's patterns reach one directory level below the deepest file it scans today.
`pattern/PATTERN-test-speed.md` states that rule under "Narrow tests carry `//lym:watch`", with its bound:
a file nested deeper than the patterns reach does not run the guard in a narrowed gate, and the whole gate still does.

## Guard tests that got patterns

The stencil guards in `contracts/stencils` share one stencil set, written `S` below:
`contracts/stencils/*.go`, `contracts/stencils/*.md` down to `contracts/stencils/*/*/*/*.md`, `contracts/orchrole/*`, `internal/stencil/*.go` and `internal/stencilstore/*.go`.
`stencils.go` embeds the stencils, `registry_test.go` holds the shared helper, and `stencilstore` serves the registry.

| Test | Patterns |
|---|---|
| `TestStencils_OpeningStencilsRenderParentDirective` | `S`, `internal/parentdirective/*.go`, `internal/editdirective/*.go` |
| `TestStencils_OpeningStencilsRenderEditDirective` | `S`, `internal/parentdirective/*.go`, `internal/editdirective/*.go` |
| `TestStencils_NoEditRuleRestatement` | `S` |
| `TestStencils_NoUnboundedAskOperator` | `S` |
| `TestStencils_NoSkillLoadInstructions` | `S` |
| `TestStencils_NameGoValues` | `S`, `internal/gatecli/*.go`, `internal/shuttleengine/*.go`, `internal/shedrun/*.go`, `internal/websterengine/*.go` |
| `TestTwillStencils_HostNeutral` | `S` |
| `TestHubGenericStencils_NameNoGoTiers` | `S` |
| `TestRefKindScan_RealTreeClean`, `TestPlanOpScan_RealTreeClean` | `internal/planparser/*.go`, `internal/planparser/*/*.go`, `internal/planglyph/*.go`, `internal/planglyph/*/*.go`, scankit |
| `TestBannedDeclarations_CliPackagesCallIntoCliwire` | `internal/webstercli/*.go`, `internal/webstercli/*/*.go`, `internal/burlercli/*.go`, `internal/burlercli/*/*.go`, scankit |
| `TestEnforcement_ScankitStdlibOnly` | `internal/testkit/scankit/*.go` |
| `TestBoardGuard_NoRawGitImportOrShellOut` | `internal/boardengine/*.go`, `internal/boardengine/*/*.go`, `internal/boardengine/*/*/*.go`, scankit |
| `TestCwdMutation_MigratedFilesStayChdirFree` | the six subject-set files by exact path, scankit |
| `TestCwdMutationGuard_NotVacuous` | `internal/fabricengine/coalesce_integration_test.go`, scankit |
| `TestNoDestructiveBypass_FabricengineProductionSource` | `internal/fabricengine/*.go`, `internal/fabricengine/*/*.go`, scankit |
| `TestNoUncontainedWrite_FabricengineProductionSource` | `internal/fabricengine/*.go`, `internal/fabricengine/*/*.go`, scankit |
| `TestMutationRecord_FabricengineProductionSource` | `internal/fabricengine/*.go`, scankit |
| `TestRemoteRefsLock_FabricengineFetchesWritingTrackingRefsGoThroughGitrepo` | `internal/fabricengine/*.go` (a shallow walk), scankit |
| `TestGitrepoBoundary_PinnedRunCallSites` | `internal/gitrepo/*.go` (a shallow walk), scankit |
| `TestNoRawGitMutation_WebsterProductionSource` | `internal/websterengine/*.go`, `internal/websterengine/*/*.go`, scankit |
| `TestVerifiedTree_PlanVerifySitesCallVerify` | the four plan-verify sites by exact path, scankit |
| `TestNoDeniedRecovery_ScannedSurfaceHasNone` | `contracts/specs/*` and `/*/*`, `contracts/stencils/*` to `/*/*/*/*`, `plugins/ly/skills/*` to `/*/*/*`, `internal/webster*`, `internal/shed*` and `internal/loom*` at `/*.go` and `/*/*.go`, `internal/shellguard/*.go`, scankit |
| `TestWebsterRetiredSteps_ScannedSurfaceHasNone` | `contracts/specs/*` and `/*/*`, `contracts/stencils/webster/*` and `/*/*`, `internal/webster*/*.go` and `/*/*.go`, `cmd/lym/norecoverydeny_test.go` (its token matcher), scankit |
| `TestSandboxCoverage_AllModulesCoveredOrExcluded` | `tools/sandbox/*SUITE.md`, `cmd/lym/main.go`, `internal/*cli/*.go`, `internal/shortcutcli/*`, scankit |

`TestCSharpFixtures_CompileWithDotnet` already carried `//lym:watch *.cs` and is unchanged.

## Guard tests left unmarked

| Test | Why it stays unmarked |
|---|---|
| `TestAgentFiledIssues_NoCreateIssueOutsideSelfreport`, `TestFslinkOnly_NoRawLinkCalls`, `TestGHGuard_NoShellOutOutsideGithubclient`, `TestPullRequestGuard_OnlyLandingAndTeardown`, `TestInteractiveSpawn_NoHeadlessClaude`, `TestSpawnRole_NoInlineRoleLiteral`, `TestTolerantRecords_NoStrictJSONDecode`, `TestConfigStrictness_PinnedCallSiteSets` | scan every non-test Go file of the module |
| `TestBareTemp_NoCreationInTheBareSystemTempDirectory`, `TestGofmt_ModuleFilesAreFormatted`, `TestNoCgo_NoFileOfTheModuleImportsC` | scan every Go file of the module, test files included |
| `TestCheckedCallInvariant_RawSitesMarkedAndPinned`, `TestGlyphConversionChokepoint_NoLocalConversion`, `TestDaemonWakeups_UnboundedWaitsStayAboveTheFloor`, `TestShuttleStop_RemoveStrandOnlyInAllowedPackages`, `TestSpawnObservability_ProductionSpawnsAreLogged`, `TestVerifiedTree_NoOtherVerifyRunCaller`, `TestVerifiedTree_NoRetiredMarker` | scan every non-test Go file under `internal/` and `cmd/` |
| `TestRegistration_AllModulesRegistered` | scans every non-test Go file under `internal/` for a `Command()` that `main.go` must register |
| `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain`, `TestLLMTier_OnlyLLMFilesReachAnLLM`, `TestTierPurity_UntaggedTestsSpawnNothing`, `TestTierTags_OneTierPerTestFile`, `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` | scan every test file of the module |
| `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt` | runs `go list -test -deps` over every package, so any import anywhere can fail it |
| `TestRegistration_MatchesDeclarers` (`internal/configreg`) | scans every non-test Go file under `internal/` |
| `TestProviderSeamImportRule`, `TestDriverChoiceSingleSiteInvariant_OnlyLoomcliReadsTheSeedDriverField`, `TestDeriveCallerSet_CliwireOnly`, `TestEnforcement`, `TestEnforcement_GeometryLiterals`, `TestEnforcement_LocationLiterals` | scan every non-test Go file of the module |
| `TestEnforcement_PairTeardownChokepoint` | scans every Go file of the module, test files included |
| `TestCopyRepoCallerSet_LymcwdOnly` | scans every Go file under `internal/` and `cmd/` |
| `TestEnforcement_TestkitInvariant` | scans every non-test Go file under `internal/`, `cmd/` and `tools/` |
| `TestEnforcement_BoltDirNameDeclaredOnce` | scans every Go file of the module, plus the stencil, spec and plugin markdown |
| `TestEnforcement_FabricVocabulary` | scans every `.go`, `.md` and `.yaml` file under the doc roots and the whole Go tree |
| `TestEnforcement_MarkdownLinks` | checks link targets that come from the link text, so a rename or deletion anywhere can fail it |
| `TestSeamMarkers_ReflectiveTestsAreMarked` | scans every test file of the module, plus the struct declarations its markers name |

## Estimated skips

The estimate replays the card changes of 13 of the report's 14 runs.
The 14th, `decision-record-on-board`, archived no plan cards.
Each card's changed files are the paths its plan card lists under Create, Edit, Delete and Extends.
An entry that names a symbol, such as `internal/slackengine#Setup`, counts as a change to every file of that package, test files or non-test files by the symbol.
A `--card` scope diffs against the merge base with the parent branch, so each card's set is the union of its own and every earlier card's changes in the run.
The replay matched those sets with `impactset.UnwatchedTests`, the matcher the gate itself uses, against the markers of this change.

- Over 214 cards, the 25 newly watched guards would have run 5,350 times, and 3,404 of those runs (64%) are skipped.
  That is 25% of the 63 tier-0 guard tests' runs per card gate.
- No guard run is skipped whole.
  Each tier keeps unmarked guards: tier 0 keeps 38 and tier 1 keeps `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt`.
  So the 787 guard runs of the report stay 787, each running fewer tests.
- `contracts/stencils` leaves the tier-0 guard run on 86 of the 214 cards, and `internal/planparser` on 197.
- Per test, the skips range from 54 of 214 cards for `TestSandboxCoverage_AllModulesCoveredOrExcluded`,
  which watches every `internal/*cli` package, to every card for `TestEnforcement_ScankitStdlibOnly`.
  The stencil guards are skipped on 141 cards (86 for `TestStencils_NameGoValues`), and the `internal/fabricengine` guards on 119.

The time saved is small, and the records can't measure it.
Over the 14 runs, the card gates' 412 tier-0 guard runs averaged 8.1 s of wall time and the 375 tier-1 runs 9.9 s.
In the tier-0 runs, the `cmd/lym` test binary took 7.0 s on average.
The two packages that drop out took 0.07 s (`contracts/stencils`) and 0.05 s (`internal/planparser`).
The skipped `cmd/lym` guards each walk one directory or read a few files.
The whole-module scans that dominate the binary's time stay, as do tier 1's `go build ./...` and `go vet`.
The gate run records time each package, not each test, so the share of the 7.0 s these skips remove is not in the records.

## Design decisions

- Patterns include `internal/testkit/scankit/*.go` for every watched guard that walks or allowlists through it.
  A scankit change can then still fail those guards in the card gate that made it.
- A recursive walk gets one directory level below the deepest file it scans today, such as `internal/fabricengine/*/*.go` for a flat package.
  A shallow walk gets only its own directory.
- A guard that reads pinned files names them by exact path, so it runs only when one of those files changes.
- `TestStencils_NameGoValues` watches the whole directory of each package declaring a value it names, since the declaration can move between that package's files.
- `TestSandboxCoverage_AllModulesCoveredOrExcluded` watches `internal/*cli/*.go`, because each module's top-level command takes its name from its own `*cli` package.
  A module whose command name came from another package would need its pattern added.
- The watched tests' names are unique in the module, so the gate's name-matched `-skip` skips no test of the same name elsewhere.
