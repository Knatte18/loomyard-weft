# Card 5 — Build the engine set with a given hook runner and sandbox probe

**Create:**
- `plan:internal/shuttleengines#LaunchSeams` -> `type LaunchSeams struct`
- `plan:internal/shuttleengines#NewWithSeams` -> `func NewWithSeams(cfg shuttleengine.Config, dirs configengine.Dirs, seams LaunchSeams) (shuttleengine.EngineSet, error)`
- `plan:internal/shuttleengines#TestNewWithSeams_CodexTakesTheGivenSeams` -> `func TestNewWithSeams_CodexTakesTheGivenSeams(t *testing.T)`

**Edit:**
- `internal/shuttleengines#registry`
- `internal/shuttleengines#New`
- `internal/shuttleengines#build`
- `internal/shuttleengines#keepWarmWith`
- `internal/shuttleengines#KeepWarm`
- `internal/shuttleengines#SessionNamers`
- `internal/shuttleengines#RecordReaders`
- `internal/shuttleengines#ToolBoundaryRenderer`
- `internal/shuttleengines#BatchingNudger`
- `internal/shuttleengines#EditRecorder`
- `internal/shuttleengines#TestKeepWarm`
- `internal/shuttleengines#TestNew`
- `internal/shuttleengines/doc.go`

**Extends:**
- `//pattern/PATTERN-shuttle-provider-seam.md`

**Uses:**
- `internal/shuttleengines#newCodexLaunchFixture`
- `internal/shuttleengine/codexengine#Seams`

**Intent:**
D10's verb and the parity test run each engine's `Prepare` with a sandbox probe and a hook runner of their own; `PATTERN-shuttle-provider-seam` lets only `internal/shuttleengines` construct a concrete engine, so the seams enter there.

`LaunchSeams` holds `RunHook shuttleengine.HookRunner` and `SandboxProbe func(bin, home, dir string) error`.
The registry's `constructor` type takes the seams as a third parameter: the Codex entry passes `seams.RunHook` (falling back to `claudeengine.RunPluginHook` when nil) and `seams.SandboxProbe` (nil keeps Codex's own probe) into `codexengine.Seams`; the Claude entry ignores them.
`build` takes the seams and hands them to each constructor.
`New` calls `build` with zero `LaunchSeams`, so production launches are unchanged; `NewWithSeams` builds the same set with the given seams.
`keepWarmWith`, `KeepWarm`, `SessionNamers`, `RecordReaders`, `ToolBoundaryRenderer`, `BatchingNudger` and `EditRecorder` call their constructors with zero seams, and the constructor literals in `TestKeepWarm` and `TestNew` take the new parameter.
Bound: the seams change only what a set built by `NewWithSeams` runs; nothing in a real launch path calls it.

`pattern/PATTERN-shuttle-provider-seam.md`'s bullet on `codexengine.Seams` gains a sentence: `shuttleengines.NewWithSeams` is the one entry point that builds the set with a caller's hook runner and sandbox probe, for `lym shuttle manifest` and the parity test.

Test, tier0, setting `CLAUDE_CONFIG_DIR` and `HOME` through `newCodexLaunchFixture`'s fixture, so not parallel:
`TestNewWithSeams_CodexTakesTheGivenSeams` builds the fixture, then rewrites its worktree codex.yaml layer to `sandbox_mode: auto` and writes into its Claude config directory an `installed_plugins.json` naming one user-scope plugin whose `hooks/hooks.json` holds one session-start command hook.
It builds the set with `NewWithSeams`, a hook runner that records its calls and prints a context, and a probe that records its calls;
`Prepare` of the Codex engine calls both, and its launch carries the runner's context.
The nil-seam fallbacks are not reached at tier0: `claudeengine.RunPluginHook` and Codex's own probe spawn `sh` and `codex`.

**ImpactSummary:** The package-private constructor type gains a parameter: every constructor call site and literal in `shuttleengines` and its two tests changes, and every exported entry point keeps its signature.

**Commit:** 5: Build the engine set with a given hook runner and sandbox probe
