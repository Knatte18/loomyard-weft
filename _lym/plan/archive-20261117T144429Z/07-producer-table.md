# Card 7 — The producer table and the scan for unlisted launch-spec literals

**Create:**
- `plan:internal/shuttleparity#producer` -> `type producer struct`
- `plan:internal/shuttleparity#producers` -> `func producers(t *testing.T, fx parityFixture) []producer`
- `plan:internal/shuttleparity#unlistedLiterals` -> `func unlistedLiterals(t *testing.T, base string, listed map[string]bool) ([]string, int)`
- `plan:internal/shuttleparity#TestProducers_CodexManifestHoldsClaudes` -> `func TestProducers_CodexManifestHoldsClaudes(t *testing.T)`
- `plan:internal/shuttleparity#TestLaunchSpecLiterals_AreListed` -> `func TestLaunchSpecLiterals_AreListed(t *testing.T)`

**Uses:**
- `plan:internal/shuttleparity#newParityFixture`
- `plan:internal/shuttleparity#parityFixture`
- `plan:internal/shuttleparity#PrepareEach`
- `plan:internal/shuttleparity#Compare`
- `plan:internal/shuttleparity#Unmet`
- `plan:internal/shuttleparity#seatTables`
- `internal/sheddriver#tacklerSpec`
- `internal/treadleengine#runJudgeCall`
- `internal/treadleengine#runTargeting`
- `internal/websterengine#FixPlanSpawnOrAttach`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#Run`
- `internal/samplerengine#SessionSpec`
- `internal/landingshed#PRReview.sessionSpec`
- `internal/loomcli#driverSpec`
- `internal/dobbyengine#TidySpec`
- `internal/mergeresolve#buildConflictSpec`
- `internal/shedarbiter#Spec`
- `internal/weaverengine#attemptSpec`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#Bouncer.RenderSeedPrompt`
- `internal/shedadapters#Bouncer.RenderJudgePrompt`
- `internal/gatefixer#SpawnOrAttach`
- `internal/frictionengine#buildReflectionSpec`
- `internal/seatengine#seatSpec`
- `internal/shedrecipe#multiLLMEntry`

**Intent:**
D1's producer table and scan, test code in `internal/shuttleparity`.

`producer` holds `Function`, the glyph of the production function holding a launch-spec literal (such as `internal/sheddriver#tacklerSpec`), and `Specs func(engine string) (shuttleengine.Spec, error)`.
`producers` returns one entry per such function outside `seatengine`: each renders the producer's prompt for the engine through the producer's own render function on `fx`'s seeded stencils with stand-in inputs, and builds the spec with the literal's role, config role, skills, `ForkSubagents` and `Requires`.
The renderers are `RenderTacklerPrompt`, `RenderCirclingJudgePrompt` (the treadle judge), `RenderTargetingPrompt`, `RenderPlanFixPrompt`, `RenderRecoveryPrompt`, `RenderMasterPrompt`, samplerengine's `RenderSessionPrompt`, `RenderPRReviewPrompt`, `RenderDriverPrompt`, `TidyPrompt`, `RenderConflictPrompt`, shedarbiter's prompt renderer in `call.go`, `RenderBatchPrompt`, `Bouncer.RenderSeedPrompt`, `Bouncer.RenderJudgePrompt` on a `NewBouncer` of stand-in config, gatefixer's prompt renderer in `prompt.go`, and `RenderReflectionPrompt`.
`judgeCall` and `runSeedSpawn` already render through `RenderJudgePrompt` and `RenderSeedPrompt`, methods taking the engine, so D1's split of an inline render needs no production change.

`unlistedLiterals(base, listed)` walks production Go files under base with `scankit`, a walk of its own since `shuttleengine`'s prompt-engine scan lives in that package's test files, and returns `file:line: <glyph>` for each unlisted literal, plus the file count:
- a `shuttleengine.Spec` composite literal that sets `Prompt` and whose enclosing function's glyph `listed` does not hold; a literal that sets no `Prompt` is skipped (the bouncer's `awaitLiveJudge` and `awaitLiveSeed` probes, and the empty specs of error returns);
- a `seatengine.Table` composite literal with at least one element whose enclosing function's glyph `listed` does not hold; an empty `seatengine.Table{}` is an error return's zero value and is skipped.
A method's glyph is `<dir>#<Receiver>.<Method>`; a literal inside a function literal takes its enclosing named function's glyph.
Skipped places: `internal/orchcli` and `internal/orchengine` (the orch spec names the Claude engine), `internal/shuttlecli` (`lym shuttle run` is hand-written and `lym shuttle open` carries no role), and `internal/testkit`.
The real module's `listed` set holds every `producers` function, every `seatTables` entry's `Builder` and `Holders`, `internal/seatengine#seatSpec` (each seat's spec, which the seat catalog covers), and `internal/shedrecipe#multiLLMEntry` (its seats come from an operator-written row config, card 6's bound).
Each failure message ends with the way forward: add the function to `producers` or to a `seatTables` entry's `Holders`, or, for a probe, set no `Prompt`.

Tests, tier0:
- `TestProducers_CodexManifestHoldsClaudes`, `//lym:guard` since it renders producers over the real stencil set:
  for each `producer` whose Codex spec `Unmet` leaves empty, it runs `PrepareEach` on `newParityFixture`'s fixture and fails naming the function and each item `Compare` returns, with card 6's way forward;
  Merriam's `websterengine#Run` spec requires forks, so it is logged with its unmet requirement and not compared.
  Not parallel, for the fixture's environment and passage reader.
- `TestLaunchSpecLiterals_AreListed`, `//lym:guard`, table rows over temp-dir sources: an unlisted Spec literal setting `Prompt` is reported with its function's glyph; a listed one is not; a Spec literal setting no `Prompt` is skipped; literals under `internal/shuttlecli`, `internal/testkit`, `internal/orchcli` and `internal/orchengine` are skipped; an unlisted non-empty `seatengine.Table` literal is reported; an empty `seatengine.Table{}` is skipped;
  a `RealModule` row scans the module against the real `listed` set and fails naming each unlisted literal, with `scankit.RequireFloor` on the file count.

**Commit:** 7: List every launch-spec producer for the parity test
