```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: no BLOCKING or MEDIUM findings; one LOW. I ran each added row and assertion step by step against the production code.

**LOW: the fakes ignore the choice, so a call site that passes the wrong choice to `EngineName` still passes**
- Changed text: `fakeShuttle.EngineName`, `agentFake.EngineName`, `tidyShuttleFake.EngineName` and `fakeDriverStarter.EngineName`.
- Each now returns a fixed `engine` field, or "claude" when unset, whatever choice it receives.
- The review's fix said to make the fake return the choice's own engine.
- Defects the new rows do catch: a call site hard-coding the engine in the `Prompt` or `Spec` call, and the codex copy not reaching `PromptEngine` or the rendered directive.
- Defect they miss: a call site passing the wrong `ModelChoice` to `EngineName`. The fixed fakes return the same answer for any choice.
- `fakeDriverStarter` used to return `choice.Engine`. The new `engine` override takes precedence, so the codex row no longer depends on the choice either.
- The earlier tests in that file still use the choice-derived path.

Per-path checks, none of which failed:
- **`shedarbiter` spawn row.** `settleFresh` calls `Prompt(..., engine, ...)` and `Spec(prompt, ..., engine, ...)` with `engine` from the fake. `spec.PromptEngine` is "codex" and the prompt carries the codex copy. A hard-coded "claude" would fail both assertions. The new assertions also run for every claude row with `claude` and `codex` as the pair, which is valid because `seededStencils` ships both copies.
- **`shedarbiter` attach row.** `attachAndConclude` builds `Spec("", ..., c.deps.Shuttle.EngineName(...), ...)`. `attaches[0].PromptEngine` is checked after the existing `Fatalf` on the attach count, so the index is safe.
- **`sheddriver` spawn row.** `settleFresh` renders through `RenderTacklerPrompt` (`tackler.go:247`) and sets `PromptEngine` in `tacklerSpec` (`tacklerprompt.go:121`). `stencilkit.Seed(t)` is a fresh directory with identical content to the one in `deps()`, so the directive comparison holds.
- **`sheddriver` re-attach row.** The `attachSpec.PromptEngine` assertion relies on the attach path building its spec through `tacklerSpec` with `engine` (`tackler.go:191`). I saw that the engine is resolved there but did not read the line that builds the attach spec, so I did not confirm it.
- **`dobbyengine` row.** `spawnTidy` resolves the engine, renders `TidyPrompt` and builds `TidySpec` with it (`tidy.go:191-195`, `:96`). `h.d.deps.StencilsDir` is the seeded directory. The test fails if the engine is hard-coded.
- **`loomcli` row.** `startLLMDriverArm` calls `c.driverStarter.EngineName(choice)`, then `RenderDriverPrompt`, then `driverSpec(prompt, engine, ...)`. `driverprompt.go:63` uses `editdirective.Directive` with that engine, and `SeedInto` ships the codex copy. The assertions are sound.
- **`gatecli` row.** `detachfixer.go:259-277` renders with `Engine: kit.engine` and sets `PromptEngine: kit.engine`. `gatefixer/prompt.go:80` reads the engine's directive from `in.StencilsDir`. `newFixerRig(t, 1, 0)` hands the red call to the fixer once, then the re-run passes. `rig.stencil` is the same seeded directory. Setting `engine: "claude"` in the constructor keeps the older rows as they were.
- **F3 message renames.** They affect strings only.
- **Not covered, outside this lens.** `cardfixer.go:199` (`engine: rig.engines.EngineName(model)`) is still not exercised. The `gatecli` row pins only the consumer, `detachfixer.go`, and not the derivation. The fixer should report F1 item 5 as partial.

Extra tool calls (1):
- One `grep` for `editdirective.` and `PromptEngine` across `gatefixer`, `loomcli` and `sheddriver`, to confirm each renderer reads the directive for the engine it is given and that `tackler.go` does not set `PromptEngine` itself.
