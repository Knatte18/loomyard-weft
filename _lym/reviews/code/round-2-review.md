---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shedarbiter/call_test.go:48
    summary: Five launch paths resolve the engine through their namer, but no test drives them under a non-Claude namer, so a Codex launch's prompt and PromptEngine are unpinned there
    fix: "In each launch test below, make the fake namer return the choice's own engine (or codex), run the launching function with a codex model choice, and assert both the started spec's PromptEngine and stencilkit.RequireEditDirectiveOf(t, stencilsDir, startedSpec.Prompt, \"codex\", \"claude\"), adding a row to the existing table test rather than a new test: (1) internal/shedarbiter/call_test.go fakeShuttle.EngineName, through Call.settleFresh; (2) internal/sheddriver/tackler_test.go agentFake.EngineName, through TacklerCall.settleFresh; (3) internal/dobbyengine/tidy_test.go tidyShuttleFake.EngineName, asserting the spec spawnTidy hands Start in TestCountLandingsAndTidy; (4) internal/loomcli/start_driver_test.go, asserting fakeDriverStarter.gotSpec.PromptEngine equals the choice's engine and the prompt carries that engine's copy; (5) internal/gatecli/cardfixer_test.go or the newCardFixer test, asserting the kit's engine is rig.engines.EngineName(model) for a choice naming another engine."
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:55
    summary: The change description says production wiring changed only in loom's driver starter and beam's board-tidy shuttle, but sheddriver.RunnerShuttle and gatecli's card fixer kit gained engine resolution too
    fix: "In _lym/landing/summary.md, Design decisions, second bullet: replace the sentence ending in loom's driver starter and beam's board-tidy shuttle with one that names every adapter that gained EngineName or resolves the engine itself: loom's driver starter (internal/loomcli/driverlaunch.go), beam's board-tidy shuttle (internal/beamcli/dobbydeps.go), sheddriver's RunnerShuttle (internal/sheddriver/agentshuttle.go) and the gate card fixer kit (internal/gatecli/cardfixer.go, through rig.engines)."
  - id: F3
    severity: NIT
    class: consistency
    location: internal/sheddriver/tacklerprompt_test.go:101
    summary: TestTacklerPrompt's failure messages still name tacklerPrompt() after the function was renamed RenderTacklerPrompt
    fix: "In internal/sheddriver/tacklerprompt_test.go, replace every tacklerPrompt() in a t.Fatalf, t.Errorf or t.Error message (lines 101, 106, 110, 115, 123, 127) with RenderTacklerPrompt()."
---

### [MEDIUM] Launch paths are unpinned under a non-Claude namer

**Location:** internal/shedarbiter/call_test.go:48, internal/sheddriver/tackler_test.go:41, internal/dobbyengine/tidy_test.go:33, internal/loomcli/start_driver_test.go:75, internal/gatecli/cardfixer_test.go:213

**Issue:** The focus asked whether each producer test asserts that the rendered prompt holds the edit-directive copy of the engine its launch resolves to, under a Codex namer.
Most producers are pinned at the launch path:
treadleengine (`runCircling` and `runMilestone` under a codex choice), samplerengine, landingshed, frictionengine, mergeresolve, shedadapters (the codex-models row checks `GotSpec.PromptEngine` against the fake's resolution), weaverengine (each rung's engine), websterengine (master, recovery, plan and verify fixers on `shuttlefake.EngineName`) and loomcli's row fixer (`choiceEngineNamer`).
Five launch paths are pinned only at their render function, never at the call that picks the engine:

- `shedarbiter.Call.settleFresh` and `attachAndConclude` call `c.deps.Shuttle.EngineName(...)`, but `fakeShuttle.EngineName` returns `"claude"` whatever the choice, and only `Prompt(dir, "codex", ...)` is tested directly.
- `sheddriver.TacklerCall.settleFresh` and `attach` work the same way: `agentFake.EngineName` returns `"claude"`, and only `RenderTacklerPrompt` sees codex.
- `dobbyengine.spawnTidy`: `tidyShuttleFake.EngineName` returns `"claude"`, and no test asserts the `PromptEngine` of the spec handed to `Start`. Only `TidyPrompt` and `TidySpec` are called directly.
- `loomcli.startLLMDriverArm`: `fakeDriverStarter.EngineName` returns the choice's engine, but no assertion reads `gotSpec.PromptEngine` or the prompt's directive copy. The tests seed a `"fake"` copy and stop there.
- `gatecli.newCardFixer` derives `engine: rig.engines.EngineName(model)`. The test's kit stub hard-sets `engine: "claude"`, so the derivation is never exercised.

Suppose one of these call sites passed a literal, or the wrong choice, to `EngineName`.
Every test would still pass, and the fault would show only as Prepare's refusal of a live Codex launch.
That is the behavior the acceptance criterion "every producer's prompt for a Codex launch holds the Codex edit directive" names, and it is a behavior the diff adds with no coverage.
The fake engine in `internal/testkit/shuttlefake` does not run `CheckPromptEngine`, so the runner-level tests do not catch it either.

### [LOW] The change description understates where production wiring changed

**Location:** _lym/landing/summary.md:55

**Issue:** The description says: "Producers get `EngineName` from the shuttle seam they already hold, so production wiring changes only where an adapter wraps the runner: loom's driver starter and beam's board-tidy shuttle."
The diff also adds `RunnerShuttle.EngineName` in `internal/sheddriver/agentshuttle.go:67`, the adapter the arbiter and tackler drive.
It also adds engine resolution to the gate card fixer kit in `internal/gatecli/cardfixer.go:199`.
The claim is therefore not borne out by the diff.

### [NIT] Stale function name in tackler prompt test messages

**Location:** internal/sheddriver/tacklerprompt_test.go:101

**Issue:** The diff renames `tacklerPrompt` to `RenderTacklerPrompt` and updates the calls, but the failure messages at lines 101, 106, 110, 115, 123 and 127 still print `tacklerPrompt()`.
`driverprompt_test.go` updated its messages in the same rename, so the two tests now disagree.

## Rejected

- Focus 1, orch engine: `orchEngine = "claude"` in `internal/orchcli/remotecontrol.go` is the only engine literal outside an engine package, and the record's "The orch's Claude pin stays a code literal" entry (parent, 2026-11-17) makes it a recorded exception to PATTERN-code-conventions.
  `orchModelChoice` and `orchSpec` both read it, so the spec's engine, its prompt engine and the remote-control check agree. Verdict: not a defect.
- Focus 2, tidy runner build error: `runnerTidyShuttle.EngineName` builds the runner under the same `once` as `Start`, and it returns the build error before any stencil is read.
  `spawnTidy` wraps it with `%w` before calling `TidyPrompt`, so a missing stencil cannot mask it, and the new subtest checks it with `errors.Is`. Verdict: not a defect.
- Focus 3, stencilkit: the block name is declared once, as `editdirective.EngineBlock`, and stencilkit and the stencils guard both reference it.
  `editdirective_test.go` moved to `package editdirective_test`, and the internal-package `testmain_test.go` imports only tmuxkit, so stencilkit's import of editdirective forms no cycle. `go vet` on both packages passes. Verdict: not a defect.
- Focus 5, driver prompt: `RenderDriverPrompt` is exported, takes the engine, and needs no session, and the summary's sentence about it matches the code. Verdict: not a defect.
- `TestCheckPromptEngine_RefusesAPromptRenderedForAnotherEngine` overlaps the engines' `TestPrepare_PromptEngine` rows.
  It is not redundant, because it alone asserts the way-forward and model-setting fragments of the refusal.
- Seatengine (focus 4): the round-1 fixer disputed the seatengine part of F4, and I accept the dispute.
  `internal/seatengine/prompt_test.go:315-316` pins a codex assistant's `apply_patch` copy and a claude assistant's Edit-or-Write copy, each with the other copy absent, through `editdirective.Directive(geom.StencilsDir, seat.stencilEngine(...))`.
  This round's F1 is the residue of round-1 F4 at five launch paths whose fakes still ignore the choice. It does not re-raise the seatengine part.
