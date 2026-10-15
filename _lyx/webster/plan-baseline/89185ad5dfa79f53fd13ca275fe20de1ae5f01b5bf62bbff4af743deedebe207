# Card 9 — engine-scrub-keys

**Create:**
- `plan:internal/reedengine#EnvScrub` -> `type EnvScrub struct`
- `plan:internal/reedengine#EnvScrub.Apply` -> `func (s EnvScrub) Apply(environ []string) (clean []string, stripped []string)`
- `plan:internal/reedengine#testScrub` -> `var testScrub EnvScrub`
- `plan:internal/reedengine#TestEnvScrub_Apply` -> `func TestEnvScrub_Apply(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#ScrubbedEnv` -> `func ScrubbedEnv() (names []string, prefixes []string)`
- `plan:internal/shuttleengines#EnvScrub` -> `func EnvScrub() reedengine.EnvScrub`
- `plan:internal/shuttleengines#scrubOf` -> `func scrubOf(entries map[string]entry) reedengine.EnvScrub`
- `plan:internal/shuttleengines#TestEnvScrub_MatchesTmuxkitServerEnviron` -> `func TestEnvScrub_MatchesTmuxkitServerEnviron(t *testing.T)`
- `plan:internal/testkit/tmuxkit#ServerEnviron` -> `func ServerEnviron(environ []string) []string`

**Delete:**
- `internal/reedengine/env.go`
- `internal/reedengine/env_test.go`
- `internal/testkit/tmuxkit#serverEnviron`
- `internal/testkit/tmuxkit#TestServerEnviron_MatchesReedServerSpawn`

**Edit:**
- `internal/reedengine#Engine`
- `internal/reedengine#New` -> `func New(cfg Config, geom Geometry, scrub EnvScrub) *Engine`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#TestServerBootEnv_ExcludesTraceID`
- `internal/reedengine#TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm`
- `internal/reedengine#newTestEngine`
- `internal/reedengine#TestWithOpLock_PathIsUnderDotLyx`
- `internal/reedengine#TestEngine_SocketAndSessionName`
- `internal/reedengine#TestWithTryOpLock_ToldGeometryValidationFailureLeavesTheLockFileUntouched`
- `internal/reedengine#newColdScratchEngine`
- `internal/reedengine#newReviveHub`
- `internal/reedengine#newHubEngine`
- `internal/reedengine#newIntegrationEngine`
- `internal/reedengine#TestMouseBootIntegration_NoLiveToggleWithoutRestart`
- `internal/reedengine#TestRemoveStrand_SoleStrandEmptiesSessionSucceeds`
- `internal/reedengine#TestDeadSelvagePaneIsHealedByUpWithoutCorruptingLayout`
- `internal/reedengine/doc.go`
- `plan:internal/shuttleengines#entry`
- `internal/shuttleengines#registry`
- `internal/shuttleengines/doc.go`
- `internal/testkit/tmuxkit#startServer`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/hubgeom#reviveWorktree`
- `internal/pairteardown#Teardown.reedEngine`
- `internal/loomcli#loomCLI.stopStepRuns`
- `internal/loomcli#parentWorktreeReviewerLive`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#probeReedEngine`
- `internal/loomcli#TestStartCmd_ReachesSpawnWatchdogWithHubAndTmuxPath`
- `internal/orchcli#Command`
- `internal/orchcli#newLiveFixture`
- `internal/orchcli#TestSmokeOrch_OneFullCycle`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#newSmokeRunner`
- `internal/battencli#battenCLI.openWindowInPrimeSession`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.loadedTacklerDeps`
- `internal/tendercli#newShuttlePair`
- `internal/reedcli#worktreeDirectory`
- `internal/reedcli#enterSession`
- `internal/reedcli#Command`
- `internal/reedcli#bootWatchdogEngine`
- `internal/reedcli#TestWatchdogDaemon`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#tearDownStandaloneReed`
- `internal/treadleengine_test#TestSmokeJudgeCirclingToyFixture`
- `internal/standalonegeom_test#TestNaming_StandaloneStrandIsShortnameAndRoleWithNoParent`
- `internal/seatengine_test#TestSmokeSeatsChairAsksAdvisor`
- `internal/burlerengine_test#TestSmokeBurlerRoundToyFixture`
- `internal/burlerengine_test#newClusterSmokeEngine`
- `internal/burlerengine/doc.go`
- `pattern/PATTERN-test-isolation.md`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#stripTraceID`
- `internal/reedengine#stripAgentNameEnv`
- `internal/reedengine#ReedState`

**Intent:**
Engines declare the environment keys a spawned process must not inherit, and reed scrubs exactly the keys it is told instead of carrying Claude's names itself (D8).

`reedengine.EnvScrub` is plain data, `Names []string` (exact keys) and `Prefixes []string` (key prefixes); `Apply` returns the environ without every entry whose key, the part before `=`, equals a name or starts with a prefix, plus the stripped keys in environ order, the contract `CleanClaudeEnv` had.
It lives in a new `internal/reedengine/envscrub.go`; `env.go` and its test are deleted with `CleanClaudeEnv`.
`New` takes the scrub as a required third argument and panics when it holds no name and no prefix: a required input makes an omitted argument a compile error, and an empty set still compiles, so it is refused as a programmer error, a fail-loud wiring guard every production caller reaches with `shuttleengines.EnvScrub()`, which never returns an empty set while an engine is registered.
`Engine` keeps the scrub, and `ensureServerAndSessionLocked` runs `e.scrub.Apply(os.Environ())` where it ran `CleanClaudeEnv`, before its `LYX_TRACE_ID` and strand-name steps, and still stamps the stripped keys into `ReedState.StrippedEnv`, whose JSON key is unchanged.
The tmux server is shared by every strand, so it scrubs every registered engine's keys.

`claudeengine.ScrubbedEnv` declares Claude's keys: the name `CLAUDECODE` and the prefix `CLAUDE_CODE_`.
`shuttleengines`' registry `entry` carries each engine's scrub declaration, and `EnvScrub` returns `scrubOf(registry)`: the union over the given entries' declarations, sorted and deduplicated, as a `reedengine.EnvScrub`.
Every production caller of `reedengine.New` passes `shuttleengines.EnvScrub()`, so `hubgeom` and `pairteardown` gain an import of `shuttleengines` and none of a concrete engine (`PATTERN-shuttle-provider-seam`).
The tests outside `reedengine` pass `shuttleengines.EnvScrub()` too; `reedengine`'s own tests cannot import `shuttleengines`, which imports `reedengine`, so they pass `testScrub`, `EnvScrub{Names: []string{"CLAUDECODE"}, Prefixes: []string{"CLAUDE_CODE_"}}`.

`tmuxkit`'s copy of the filter stays, since `reedengine`'s tests import the kit, but `serverEnviron` becomes the exported `ServerEnviron` so a test outside the kit can pin it; `startServer` calls it.
The pin moves out of the kit, which gains no import of a concrete engine: `TestServerEnviron_MatchesReedServerSpawn` is deleted and `shuttleengines`' `TestEnvScrub_MatchesTmuxkitServerEnviron` asserts that over a planted environ (`CLAUDECODE`, a `CLAUDE_CODE_` key, a `CLAUDE_CODEX` key that must pass, `LYX_TRACE_ID`, `LYX_STRAND_NAME`, `LYX_PARENT`, an unrelated `LYX_` key, `HOME`, `PATH`) `tmuxkit.ServerEnviron` equals `EnvScrub().Apply`'s clean environ without the three lyx names reed's spawn strips in its own steps.

Tests:
- `TestEnvScrub_Apply` carries the deleted `TestCleanClaudeEnv`'s rows under `testScrub` (Claude keys stripped in environ order, an environ without them unchanged) and adds a row whose scrub names another key and prefix, which `Apply` strips while the Claude keys pass, so it scrubs exactly the told keys.
  It carries the deleted test's `//testtiming:keep` marker, its reason reworded to name `EnvScrub.Apply` stripping exactly the told names and prefixes.
- `TestEngine_SocketAndSessionName` gains an assertion that `New` panics over `EnvScrub{}`; its existing `New` over `testScrub` is the build half.
- `TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm` sets the process variable `LYX_REED_SCRUB_PROBE` with `t.Setenv`, which this test already may since it does not run in parallel, and `newColdScratchEngine` builds its engine over `testScrub` plus the name `LYX_REED_SCRUB_PROBE`; after the cold boot the test loads the state and asserts `ReedState.StrippedEnv` holds `LYX_REED_SCRUB_PROBE` and that every entry in it is a name or prefix of that scrub, so the server spawn scrubs the told set and records it.
- `TestServerBootEnv_ExcludesTraceID` builds its environ with `testScrub.Apply` where it called `CleanClaudeEnv`.
- `TestEnvScrub_MatchesTmuxkitServerEnviron` also runs `scrubOf` over two fake entries declaring an overlapping name and prefix, asserting the union is sorted with each name and prefix once.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengines` to 4; `internal/reedengine` stays at its row, since `TestEnvScrub_Apply` replaces `TestCleanClaudeEnv`.

Docs: `reedengine`'s `doc.go` says the server spawn scrubs the keys it is told; `shuttleengines`' `doc.go` names `EnvScrub`; `burlerengine`'s `doc.go` and `pattern/PATTERN-test-isolation.md` name the told scrub where they named `CleanClaudeEnv`; `PATTERN-shuttle-provider-seam`'s background says an engine's scrub keys reach reed only through `shuttleengines`.

**ImpactSummary:** Re-signs `reedengine.New` across every production and test constructor in the CLI packages, `hubgeom`, `pairteardown` and the smoke suites; the scrubbed key set is unchanged for Claude, and `StrippedEnv` keeps its shape.
