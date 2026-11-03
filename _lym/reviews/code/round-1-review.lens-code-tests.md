```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing the test that pins each behaviour the diff adds or changes, or none.

**Per behaviour: pinning test**

| Behaviour | Test |
|---|---|
| Hub and machine layers stack name by name, machine wins, the hub's other names stay | `TestLoadConfig_StacksHubAndMachineName` rows `HubFileAlone` and `MachineValueWinsByNameAndKeepsTheRest` |
| No `env.yaml` sets nothing, and `Vars` is never nil | Row `NoFileSetsNothing` |
| A number value reads as text | Row `NumberValueReadsAsText` |
| Names with a dash or a leading digit, or starting `LYM_`, are refused | Rows `NameWithADashIsRefused`, `NameStartingWithADigitIsRefused`, `LymOwnNameIsRefused` |
| The local layer refuses `vars` | Row `LocalLayerIsRefused`, plus the local half of `TestSet_MachineLayerTakesAVariable` |
| A mapping value under a name is refused | Row `MachineValueThatIsAMappingIsRefused` |
| `templateValueAt` judges a name under an empty per-name map as a scalar, so the machine write the issue found refused now succeeds | `TestSet_MachineLayerTakesAVariable` and the machine-layer rows |
| `Config.Entries` is in name order | `TestConfig_EntriesAndBeneath` |
| `Config.Beneath` lets own win, changes neither input, and returns a non-nil map for nil `Vars` | `TestConfig_EntriesAndBeneath` |
| `env` registered as a hub-wide module with machine key `vars` | `TestModules_SeedOnlyAndHubWideFlags` and `TestModules_EveryTemplateKeyClassified` |
| `envconfig` is a degrading caller | `configStrictnessDegradingSet` entry |
| `Strip` reports an empty mapping equal to an empty lower as removed | `TestStrip` row "empty mapping equal to an empty lower removed" |
| Gate tier: the module's variables replace inherited values, and the tier's own `env` replaces the module's | `TestGateTest_Scenario`, subtest "a tier runs with the env module's variables beneath its own env". It covers `BUILD_MARK` and `GOFLAGS`, standalone worktree only. |
| `Config.StepWithHubEnv`: the step's env wins, the config is left unchanged, an undeclared step is refused | `TestLoadConfig`, subtest "StepWithHubEnv lays the env module's variables beneath the step's own env" |
| `paneEnvExports`: no seam, empty answer, name-ordered exports with quoting, failed read exports nothing and logs the strand, in both dialects | `TestPaneEnvExports` |

**Findings**

1. MEDIUM, `design`. Behaviour with no pinning test: the shared build runs with the module's variables, and the build's own `env` wins.
   - Location: `internal/gatecli/run.go:138` and `internal/gatecli/build.go:43`.
   - The only gate test drives a tier, via `renderTierEnv` at `tierrun.go:71`. Deleting the variables from the build path leaves every test green.
   - The focus names "the shared build" explicitly.

2. MEDIUM, `design`. Behaviour with no pinning test: the env module is loaded inside a hub, with the machine layer (`fabricengine.ConfigDirs`), for gate runs.
   - Location: `internal/gatecli/run.go:405-411`.
   - The one gate test uses `standaloneWorktree`, so only the `inHub == false` branch runs.
   - A regression to `Dirs{Anchor}` alone inside a hub would drop the hub file and the machine override, and no test would fail.

3. MEDIUM, `design`. Behaviour with no pinning test: the gate refuses when the env config fails to load.
   - Location: `internal/gatecli/run.go:62-65`.
   - No test reaches the "cannot read the env config" refusal.
   - The refusal also matters for `PATTERN-refusal-way-forward`.

4. MEDIUM, `design`. Behaviour with no pinning test: Drift capture and compare processes receive the variables through the three wiring sites.
   - Locations: `internal/driftcli/cli.go:204` (`takeRun`), `internal/driftcli/cli.go:391` (`runCaptureChild`) and `internal/loomcli/driftwiring.go:106`.
   - Only the `StepWithHubEnv` unit is pinned. Reverting any call site to `cfg.Step` fails no test.
   - `w.configDirs()` in `driftcli/wiring.go` is likewise unpinned.

5. LOW, `design`. Behaviour with no pinning test: `StepWithHubEnv` returns an error when the env config fails to load.
   - Location: `internal/driftengine/config.go:184-187`.
   - The subtest covers the undeclared-step error only.

6. MEDIUM, `design`. Behaviour with no pinning test: `launchStrandLocked` puts the exports ahead of `launchCmd`, on every launch and relaunch.
   - Location: `internal/reedengine/spawn.go:119`.
   - `TestPaneEnvExports` pins only `paneEnvExports`. No test checks the composed launch line.
   - Dropping line 119, or swapping the `sh.Chain` arguments, fails nothing.

7. LOW, `design`. Behaviour with no pinning test: `hubgeom.paneEnv` loads the hub's env config with the machine override and returns `Vars`.
   - Location: `internal/hubgeom/hubgeom.go:464-470`, wired at line 90.
   - Nothing asserts `PaneEnv` is non-nil in hub mode or that it returns the hub and machine values.

**Test economy, NIT.** `TestLoadConfig_StacksHubAndMachineName` and `TestSet_MachineLayerTakesAVariable` each call `os.MkdirAll(dirs.Anchor, ...)`, which `hubDirs` already did at `config_test.go:~45`. The final `LoadConfig` assertion in the `Set` test repeats the stack test's machine-wins coverage. No new test duplicates another behaviour.

**Tool calls:** none. The manifest context covered everything this lens needed.
