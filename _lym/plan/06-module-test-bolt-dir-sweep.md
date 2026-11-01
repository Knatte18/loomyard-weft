# Card 6 — module-test-bolt-dir-sweep

**Edit:**
- `cmd/lym#TestStencilSeedTarget_PlainRepoHasNoHub`
- `cmd/lym#uncontainedWriteAllowlist`
- `internal/adoptcli#TestCheckView`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/boardcli_test#cliFixture`
- `internal/boardcli_test#cliFixture.reset`
- `internal/boardcli_test#defaultHubBoardConfig`
- `internal/boardcli_test#seedBoardFile`
- `internal/boardcli_test#seedCwd`
- `internal/boardcli_test#seedHubBoardConfig`
- `internal/boardcli_test#stepBoardPathResolution`
- `internal/boardcli_test#stepContract`
- `internal/boardcli_test#stepErrorAndEdgeCases`
- `internal/boardcli_test#stepLoadsHubBoardConfig`
- `internal/boardcli_test#stepLookupContract`
- `internal/boardcli_test#stepSiblingWarning`
- `internal/boardcli_test#stepStrictPayloadShapes`
- `internal/boardcli_test#TestCLI`
- `internal/boardengine/boardtest#seedWikiRepo`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture`
- `internal/boardengine_test#refusingSlugRule`
- `internal/boardengine_test#TestLoadConfig_ScalarKeysAndTemplateDefaults`
- `internal/boardengine_test#TestSlugRule`
- `internal/configcli#boardHead`
- `internal/configcli#TestConfigOverRealHub`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configengine_test#TestLoad`
- `internal/configengine_test#TestLoadOrTemplate`
- `internal/configengine_test#TestLoadOrTemplate_LymDirStatFailure_DoesNotFallback`
- `internal/configengine_test#TestLoad_Refusals`
- `internal/fabriccli_test#makeCLICloneWeftBare`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`
- `internal/hubforge#assertRealHub`
- `internal/hubreconcile#TestGeometry_Paths`
- `internal/ideengine#writeBoard`
- `internal/orchcli#TestWriteHubRules`
- `internal/preflight_test#TestHealthyHub`
- `internal/preflight_test#TestResolveMode`
- `internal/reedcli#countLogsWithPrefix`
- `internal/reedcli#TestSmokeDebugLog`
- `internal/reedcli#waitForFreshServerLog`
- `internal/reedcli#writeFakeLog`
- `internal/shedcli#TestWriteSeed_RefusesFabricsOwnCheckouts`
- `internal/shuttleengine/claudeengine#TestBuildSettings_CreelMailHook`
- `internal/tendercli#TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub`
- `internal/vscode#TestBuildWorkspace`
- `internal/webstercli#TestPersistentPreRun_OpenFabricWiredButUninvoked`
- `internal/webstercli#TestWire_ModeHubSelectsHubMode`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/yamlengine#TestSetValues`

**Uses:**
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#BoltDirName`

**Intent:** Make the other packages' tests take the Bolt directory name from the constant and name the Bolt checkout in their comments (D9).

- Each `"_board"` string these members build a path, a fixture or an expected message from becomes `fabricengine.BoltDirName`, and a hand join of a hub and that name becomes `fabricengine.BoltDir(hub)`.
  `internal/yamlengine#TestSetValues` and `internal/vscode#TestBuildWorkspace` take `lymcwd.BoltDirName`, the same value, since they are in-package tests of packages `fabricengine` imports, so importing `fabricengine` there closes a cycle; every other listed member takes `fabricengine.BoltDirName`.
- Each comment in these members that spells `_board` names "the Bolt checkout" instead; `internal/boardcli/cli_test.go` and `internal/reedcli/smoke_debuglog_test.go` carry the token in their file headers, so every member of those two files is listed.
- No assertion changes meaning, since each compared value is the same string.

**ImpactSummary:** Test-only edits in about twenty packages outside fabric; compared values are unchanged, so no production code or outcome moves.
