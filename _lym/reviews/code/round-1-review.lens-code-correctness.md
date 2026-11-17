```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes, with concrete inputs and failure cases.

**Per function or branch: the case I ran, and whether the code is correct**

- `shuttleengine.CheckPromptEngine` (`internal/shuttleengine/promptengine.go:11`):
  - HandWritten=true with PromptEngine="" returns nil. Correct.
  - PromptEngine="" refuses and names the role and the engine. Correct.
  - PromptEngine="codex" on claude refuses and names both engines. Correct.
  - Role="" gives the text `role ""`, which is harmless. Correct.
- Claude and Codex `Prepare` (`claudeengine.go:126`, `codexengine.go:103`): the check runs before any config load or write, so the run directory stays empty on refusal, and `launch` removes it. Correct.
- `EngineSet.EngineName` and `Runner.EngineName` (`engineset.go:81`): an empty `choice.Engine` gives `defaultName`, and "codex" gives "codex". Correct.
- `Runner.engineFor` refactor: `rs.Engine=""` takes `EngineName`'s default, which is the same `defaultName` field `DefaultName()` returned before. Correct.
- Single `Prepare` call site (`run.go:642`): all launches, including start retries, reuse the producer's spec. The carry-over block is appended Go text, and `PromptEngine` is kept. Correct.
- Probe-only specs set no `PromptEngine` and never reach `Prepare`. These are weaver `resume.go:293`, `reset.go:317` and the seat, arbiter and tackler probes. Correct.
- `editdirective.Directive` with engine="": `EngineBlockName("", …)` resolves to `engine/edit-directive`, which is missing, so it returns an error that names it. Correct, but see F1.
- `runnerTidyShuttle.EngineName` (`beamcli/dobbydeps.go:692`): when the runner build fails and TidyModel names no engine, it returns "". `TidyPrompt` then fails on the missing `engine/edit-directive` stencil before `Start` runs. **Wrong, F1.**
- `dobbyengine.spawnTidy`, `TidyPrompt` and `TidySpec` give the same engine to the render and the spec. Correct.
- `shedadapters.RenderSeedPrompt` and `runSeedSpawn`:
  - On a template read failure, `promptRenderError` carries the same reason string as before, and the logger logs reason and cause as before. Correct.
  - Spec engine = render engine = `EngineName(Models.Seed)`. Correct.
- `shedadapters.RenderJudgePrompt` and `judgeCall`:
  - Report checks still run first.
  - On an unreadable declared PATTERN changes file, the reason carries the formatted way forward and the cause is the error, the same as before.
  - A facts-file write failure degrades as before.
  - `outputs` is recomputed identically.
  - Correct.
- `renderFailureReason` and `renderFailureCause` with a plain error: they return `err.Error()` and the error itself. Correct.
- `treadleengine.renderJudgePrompt`, `RenderCirclingJudgePrompt`, `RenderMilestoneJudgePrompt` and `runJudgeCall`:
  - Milestone's extra `hard_cap` is merged into the values.
  - OutputFiles now come from `in.VerdictPath` and `in.HandoffPath`, the same values the stencil markers carry.
  - A render failure takes the fallback verdict.
  - Correct.
- `treadleengine.RenderTargetingPrompt` and `runTargeting`: a failure returns ("", false), and the engine is threaded to the spec. Correct.
- `samplerengine.SessionSpec` and `RenderSessionPrompt`:
  - Nil Engines is refused after the required-field checks.
  - Error wrapping keeps the `sampler: SessionSpec:` prefix.
  - Correct.
- `landingshed.sessionSpec` and `RenderPRReviewPrompt`: engine from `Sessions.EngineName(choice)`, the same choice the spec uses. An error keeps its `landingshed: <name>:` prefix. Correct.
- `frictionengine.buildReflectionSpec` and `RenderReflectionPrompt`, and `mergeresolve.buildConflictSpec` and `RenderConflictPrompt`: the same choice feeds both the render and the spec. Correct.
- gatefixer:
  - `RenderPrompt`'s `in.Engine` is threaded through, and `SpawnOrAttach` copies `spec.PromptEngine` into the shuttle spec.
  - The loomcli, gatecli, weaver and webster fixers compute the engine from the same choice as `ModelChoice`.
  - Correct.
- loomcli driver: in `startLLMDriverArm`, the engine comes from `driverStarter.EngineName(choice)`, and `driverSpec` sets it. Correct.
- `seatengine`: `seatValues` and `startSeat` both use `seat.stencilEngine(geom.DefaultEngine)`. `DefaultEngine` is `shuttleCfg.Engine` in loomcli and burlercli (`burlercli/wiring.go:157,224`), the same value the engine set defaults to. Correct.
- `shedarbiter` `settleFresh`, `attachAndConclude` and `Spec`, and `sheddriver` tackler `settleFresh`, `attach` and `tacklerSpec`: both derive the engine from `Settings.ModelChoice`. Correct.
- `weaverengine.runBatch` and `fixerSpec`: the engine follows `deps.Ladder[rung]` and `deps.Ladder[fixer.Rung]`, the same choices that launch. Correct.
- websterengine Master, recovery, plan fixer and verify fixer: the engine comes from the same resolved role choice, and the recovery and Master renders were moved after the choice lookup. Correct.
- `orchcli.orchModelChoice` and `orchSpec`: Engine="claude" and PromptEngine="claude". The remote-control lookup now resolves claude whatever the default is. Correct.
- `shuttlecli` run: `HandWrittenPrompt` is set on the one spec built from both `--prompt` and `--prompt-file`. Correct.

**Findings**

F1. Severity MEDIUM, class `design`.
- Location: `internal/beamcli/dobbydeps.go:692-699` (`runnerTidyShuttle.EngineName`).
- Summary: when the shuttle runner cannot be built and the board-tidy model names no engine, `EngineName` returns "". `spawnTidy` then fails in `TidyPrompt` with a missing `engine/edit-directive` stencil error, so the runner build error is never reported. The doc comment claims "the start that follows fails with the build's error", which is false on this path.
- Evidence:
  - `internal/beamcli/dobbydeps.go:693-697` returns `choice.Engine` on `s.err`.
  - `internal/dobbyengine/tidy.go:91-94` renders before `Start` and returns the render error.
  - `internal/stencilstore/engineblock.go:25-26`: `path.Join` turns engine "" into `engine/edit-directive`.
  - `internal/shuttleengine/template.yaml:13`: a model choice that names no engine is the normal case.

**Tool calls**
1. Grepped the `DefaultEngine` writers, `Runner.start`/`launch` callers, `EngineBlockName` and the shuttle template default. My context had the seat geometry only for loomcli, not burlercli, and lacked the empty-engine path resolution.
2. Scanned production `Spec{` literals that lack `PromptEngine`, to find launch paths the diff missed. My context had only the diff's literals.
3. Read the flagged literal sites (`open.go`, weaver `resume`/`reset`, `orchcli runs`, and others) to tell launches from probes and unrelated specs.
4. Grepped `open.go` to see whether `Runner.Open` goes through `Prepare`. It does not; it uses `OpenLaunch` with an `OpenSpec`.
