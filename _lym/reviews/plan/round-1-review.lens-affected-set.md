```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, the set of readers, callers and tests it forces, and which cards cover it. Three findings, all where a card covers only part of its set.

## Per change: the set found and the cards covering it

- **Card 3: launching engine sets switch to `NewForWorktree`.**
  - Set: every `shuttleengines.New` site whose runner starts a session. That is `loomCLI.wire`, `burlerCLI.wireHub`, `wireCLI`, `shuttlecli.Command`, `battenCLI.wire`, `battenCLI.primeShuttle`, `websterCLI.wireHub`, `newStrandRig`, plus `battencli.primeShuttleRunner`.
  - `primeShuttleRunner` launches through `battencli.ShuttleRunner`, which `beamcli`'s `runnerTidyShuttle.Start` calls (`StartGated`).
  - The other sites only stop, read or send: `pairteardown`, `tendercli`, `creelcli`, `weavercli`, `loomcli` `decision.go`/`looparm.go`/`looplock.go`, `battencli` `parentlock.go`/`wire.go:711`/`cancelcmd.go`.
  - Covered by card 3, except `primeShuttleRunner`.
- **Card 3: the `constructor` type change.**
  - Set: `registry`, `build`, `New`, `NewWithSeams`, `SessionNamers`, `RecordReaders`, `keepWarmWith`, `TestKeepWarm`.
  - Covered by card 3.
- **Card 5: `adoptDeps.PickAccount`, called from `runForeground` and `takeOverRuns`.**
  - Set: `hubDepsAs`, `TestAdopt`, and the test world's `adoptDeps` builder `world.deps` (`internal/adoptcli/adopt_test.go:124-125`).
  - Card 5 lists `hubDepsAs` and `TestAdopt` but not `world.deps`.
- **Card 6: `buildLaunchCmd` and `buildResumeCmd` re-sign.**
  - Set: `Prepare` (`claudeengine.go:299,318`), `TestBuildLaunchCmd`, `TestBuildResumeCmd`, `TestBuildResumeCmd_NoticePosix` (`command_test.go:343,802,821`).
  - Covered by card 6. The `codexengine` functions of the same name are separate.
- **Card 6: `ScrubbedEnv` gains `CLAUDE_CONFIG_DIR`.**
  - Set: `EnvScrub`, `tmuxkit.ServerEnviron`, `TestEnvScrub_MatchesTmuxkitServerEnviron`.
  - The many smoke tests that pass `EnvScrub()` only pass the scrub through and assert no contents.
  - Covered by card 6.
- **Card 7: `sessionHolder`, `processLiveness`, `installedPlugins` and `sessionScopeFor` re-sign.**
  - Set: `resume.go:42,51`, `prober.go:34`, `scope.go:117`, `claudeengine.go:246`, `command.go:476`, `prober_test.go:112`, `resume_test.go:256`.
  - Covered by card 7, with `TestSessionHolder` and `TestProcessLiveness` listed.
- **Card 7: `SessionNamers` and `RecordReaders` re-sign.**
  - Set: seven production callers, which are `loomcli/wiring.go:690`, `looparm.go:164`, `battencli/wire.go:1432`, `drive.go:522`, `reedcli/watchdog.go:359`, `gatecli/cardfixer.go:130` and `tools/tokencount/main.go:341`.
  - Tests: `shuttleengines_test.go:267,310`.
  - Covered by card 7.
- **Card 8: `checkResumable`, `checkResume` and `launch` re-sign.**
  - Set: `resume.go:42`, `resume_test.go:185`, `resume_unix_test.go:26`, `run.go:604,623`, `relaunch.go:52`.
  - Covered by card 8, with `TestCheckResumable` and `TestCheckResumable_UnstattableTranscriptRefusesNamingTheStatFailure` listed.
- **Card 8: deletes `claudeProjectDirFor`, `projectsRoot` and `sessionRegistryDir`.**
  - Production callers: `resume.go`, `sessionname.go`, `background.go`, `audit.go`, `usage.go`, `transcriptread.go`, `prober.go`, covered by cards 7 and 8.
  - Test callers: `sessionname_test.go:20,78` and `audit_test.go:278`. The first two are not listed, but `caller-uncovered` is the gate's check, so no finding here.
- **Card 10: `writeOpenFiles` re-sign.**
  - Set: `OpenLaunch` (`command.go:477`).
  - Covered by card 10.
- **Card 11: `ReedOps` grows by three methods.**
  - Set: `reedengine.Engine`, `fakeReed`, `shuttlefake.Reed`, and the test types embedding `*fakeReed`: `skillReed`, `flakyStatusReed`, `keyTimesReed`, and `framedReed` through `tuiReed`.
  - `websterengine` and `gatefixer` take `ReedOps` as a field, filled with existing fakes.
  - Covered by card 11.
- **Card 12: `runIsLive` and `unfinished` re-sign.**
  - Set: `agentwait.go:52` (`ReadAgentWaits`), `activity.go:115` (`ReadAgentActivity`), `carryover.go:123,154` (`unfinished`, `carriedInstructions`).
  - Covered by card 12.
- **Card 12: `RunSessionState.Relaunching` and `AgentActivity.Relaunching` are the "reports relaunching" output.**
  - Readers outside the package: `shuttlecli/state.go` (`lym shuttle state`); `tendercli/deps.go`, which reads only runs with a live strand; and the activity mappers in `battencli`, `loomcli`, `shuttleengines` tests and smoke tests.
  - Card 12 adds the fields, but no card makes `lym shuttle state` print them.
- **Card 17: `orchengine.Session` grows by two methods.**
  - Set: `orchcli.runnerSession` (`session.go:21`) and `fakeSession`.
  - Covered by card 17.

## Findings

1. **BLOCKING, class design — card 3 (`_lym/plan/03-engine-worktree-geometry.md`).**
   - Summary: the launching-wiring set misses `internal/battencli#primeShuttleRunner`. It builds its engines with `shuttleengines.New(shuttleConfig, dirs)` and is reached through `battencli.ShuttleRunner` by `beamcli`'s `runnerTidyShuttle.Start`, which launches the dobby's board-tidy agent in the beam worktree with `StartGated`. That launch is told no worktree, so it resolves only the hub pin. A beam's tidy agent then ignores the beams pin and the beam's entry, against D4 ("at every launch it builds") and D9.
   - Fix: add `internal/battencli#primeShuttleRunner` to card 3's Edit, switching it to `shuttleengines.NewForWorktree(cfg, dirs, hubgeom.EngineWorktree(location))`.
   - Evidence: `internal/battencli/cancelcmd.go:517-541`, `internal/battencli/dobbywire.go:40-41`, `internal/beamcli/dobbydeps.go:833-839`.
2. **MEDIUM, class consistency — card 5 (`_lym/plan/05-account-pick-at-making.md`).**
   - Summary: `adoptDeps` gains `PickAccount`, which `runForeground` and `takeOverRuns` call after each `AddPair` that succeeds. The package's shared test builder `world.deps()` builds `adoptDeps` without the field, so every existing adopt test that adds a pair calls a nil func and panics. Card 5 lists only `TestAdopt`.
   - Fix: list `internal/adoptcli#world.deps` under Edit and give it a recording `PickAccount`, or have the call sites skip a nil seam.
   - Evidence: `internal/adoptcli/adopt_test.go:124-125`, `internal/adoptcli/foreground.go:124,326`.
3. **LOW, class consistency — card 12 (`_lym/plan/12-run-relaunching-record.md`).**
   - Summary: D7 says a reader that reports a run's session state reports a relaunching run as relaunching. Card 12 adds `RunSessionState.Relaunching`, but `lym shuttle state` prints only `stateFields(reading.State)` plus strand, guid and history. That verb would show a relaunching run as an empty state, not as relaunching, and no card changes it.
   - Fix: add `internal/shuttlecli` (the state verb's run map) to card 12 so it prints `relaunching`, with a row.
   - Evidence: `internal/shuttlecli/state.go:45-62`.

## Tool calls

1. Grep of the callers of the re-signed and deleted claudeengine and shuttleengine members, and the test callers of `SessionNamers` and `RecordReaders`: my context had only the production callers of some of them.
2. Grep of the consumers of `ReadAgentActivity`, `ReadSessionStates` and `ReadAgentWaits`, the `ReedOps` and `orchengine.Session` implementers, the `adoptDeps` test literals, the scrub-list tests and the claude.yaml key docs: none of these sets were in my context.
3. Grep of the shuttleengine test reed types, the `websterengine` and `gatefixer` reed fakes, the `SetWaitMark` implementations and the activity-field uses in `tendercli` and `shuttlecli`: to confirm which `ReedOps` fakes embed `fakeReed` and how the session-state readers use the fields.
4. Read of `tendercli/deps.go:398-460`, `shuttlecli/state.go:40-75` and the `tuiReed` definition: to judge whether those readers are affected by a relaunching record, and whether `framedReed` inherits the new methods.
