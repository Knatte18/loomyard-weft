```yaml
lens: fix-proposal
outcome: reported
```

Findings (proposals the fixer carried out: F1 in part, F2, F3):

1. **MEDIUM — F1 items 1–4: the engine fakes ignore the choice, so the proposal's "wrong choice" case still passes.**
   - Changed text: `fakeShuttle.EngineName` in `internal/shedarbiter/call_test.go`, `agentFake.EngineName` in `internal/sheddriver/tackler_test.go`, `tidyShuttleFake.EngineName` in `internal/dobbyengine/tidy_test.go` and `fakeDriverStarter.EngineName` in `internal/loomcli/start_driver_test.go`.
   - Summary: each fake now returns a fixed `engine` field whatever `ModelChoice` it receives.
   - Evidence:
     - The review's premise says "Suppose one of these call sites passed a literal, or the wrong choice, to `EngineName`".
     - The fix key says to make the fake return "the choice's own engine".
     - A call site that passes a zero `ModelChoice` instead of `settings.ModelChoice` (`call.go:174`, `call.go:247`, `tackler.go:191`, `tackler.go:247`) still gets `"codex"` from these fakes, so every new row still passes.
     - The new rows catch only a hard-coded `"claude"` literal.
     - `fakeDriverStarter` lost its choice-derived behaviour: it used to return `choice.Engine`, and its new `engine` override replaces that when set.
     - The arbiter fixture's `settings.ModelChoice` and the tackler's carry no `Engine`, so a choice-derived fake needs the row to set `Engine: "codex"` in the settings.

2. **MEDIUM — F1 item 5: the derivation at `internal/gatecli/cardfixer.go:199` is still unpinned.**
   - Changed text: the new `CodexKitRendersTheCodexPrompt` subtest and `fixerRig.engine` in `internal/gatecli/cardfixer_test.go`.
   - Summary: the fix key asked for the kit's engine to equal `rig.engines.EngineName(model)` for a choice naming another engine. The subtest instead sets `kit.engine` directly through the seam, so it covers only the consumer in `detachfixer.go:259-277`.
   - Evidence: the review names the derivation as the untested line (the old stub hard-set `engine: "claude"`). Changing `cardfixer.go:199` to a literal `"claude"` still passes every test. This departure from the proposal needs a "fixed differently" mark in the fixer-report, naming the proposed change not made and why. Without that mark, it reads as fixed as proposed.

3. **LOW — F3: the rename left a stale name in a stencil header.**
   - Changed text: the `tacklerPrompt()` to `RenderTacklerPrompt()` rename.
   - Evidence: `contracts/stencils/shed/shed-template-tackler.md:3` still reads "filled via internal/stencil.Fill (tacklerPrompt in internal/sheddriver)". It is the same stale name, one hop beyond the six lines the reviewer listed. The stencil carries an `lym-stencil` hash stamp, so changing it may need a restamp.

F2 has no finding. The four production adapters that gained or resolve `EngineName` (`RunnerShuttle`, `runnerDriverStarter`, `runnerTidyShuttle`, the card fixer kit) are all named in the new summary text.

Calls made:
- `grep -rn "tacklerPrompt"` over `.go` and `.md` files, to check that the F3 rename left no other stale mentions, which my context lacked.
