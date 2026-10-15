# Card 4 — Driver, arbiter, alignment and tackler settings carry a ModelChoice

**Delete:**
- `internal/loomengine#DriverSettings`

**Edit:**
- `internal/loomengine#ResolveDriver` -> `func ResolveDriver(cfg Config, reg modelspec.Registry) (shuttleengine.ModelChoice, error)`
- `internal/loomengine#ArbiterSettings`
- `internal/loomengine#ResolveArbiter`
- `internal/loomengine#AlignmentSettings`
- `internal/loomengine#ResolveAlignment`
- `internal/loomengine#TestResolveDriver`
- `internal/loomengine#TestResolveDriver_Refuses`
- `internal/loomengine#TestResolveDriver_EmptyDriverIsOffDefault`
- `internal/loomengine#TestResolveArbiter`
- `internal/loomengine#TestResolveAlignment`
- `internal/loomcli#driverSpec` -> `func driverSpec(prompt string, reportPath string, choice shuttleengine.ModelChoice) shuttleengine.Spec`
- `internal/loomcli#loomCLI.startLLMDriverArm`
- `internal/loomcli#arbiterSpec`
- `internal/loomcli#alignmentSettings`
- `internal/loomcli#TestDriverSpec`
- `internal/loomcli#TestArbiterSpec`
- `internal/loomcli#newArbiterFixture`
- `internal/alignment#Settings`
- `internal/alignment/doc.go`
- `internal/sheddriver/doc.go`
- `internal/alignment#Spec`
- `internal/alignment#TestSpec`
- `internal/alignment#sessionHarness.spawnInput`
- `internal/sheddriver#TacklerSettings`
- `internal/sheddriver#ResolveTackler`
- `internal/sheddriver#tacklerSpec`
- `internal/sheddriver#TestResolveTackler`
- `internal/sheddriver#TestTacklerSpec`
- `internal/sheddriver#newTacklerFixture`

**Intent:**
D2 and D3 for the single-session roles.
`ResolveDriver` returns the driver's `shuttleengine.ModelChoice` itself, the zero choice (default engine, provider defaults) for an empty `driver` key, and `DriverSettings` is deleted; `driverSpec` takes the choice and sets `ModelChoice: choice`, and `startLLMDriverArm` passes it.
`ArbiterSettings`, `AlignmentSettings`, `alignment.Settings` and `sheddriver.TacklerSettings` replace their `Model`, `Effort` and `Version` fields with an embedded `shuttleengine.ModelChoice`, the timeout and spawn budget staying beside it, not inside it.
`alignment/doc.go` and `sheddriver/doc.go` say the settings carry a model choice, engine included, beside the timeout.
`ResolveArbiter`, `ResolveAlignment` and `ResolveTackler` build that choice with `shuttleengine.NewModelChoice(resolved)`; `arbiterSpec`, `alignment.Spec` and `tacklerSpec` set `ModelChoice` from their settings whole; `loomcli`'s `alignmentSettings` copies the choice and the timeout.

Tests:
`TestResolveDriver`, `TestResolveArbiter`, `TestResolveAlignment` and `TestResolveTackler` assert the resolved `Engine` is the alias's engine (`claude`), the arbiter's and alignment's empty key falling back to `opus[high]` and the tackler's to `haiku` included;
`TestResolveDriver_EmptyDriverIsOffDefault` asserts the zero choice, an empty `Engine` reading as the default engine;
`TestDriverSpec`, `TestArbiterSpec`, `alignment.TestSpec` and `TestTacklerSpec` pass a choice with `Engine: "b"` and assert the spec carries it beside the model triple.
`TestResolveDriver_Refuses`, `newArbiterFixture`, `sessionHarness.spawnInput` and `newTacklerFixture` move their settings literals to the embedded choice.

**ImpactSummary:** four settings types and one deleted type; their resolvers, spec builders and tests change, every caller reading a settings field compiles unchanged through promotion.

**Commit:** 4: driver, arbiter, alignment and tackler settings carry a ModelChoice
