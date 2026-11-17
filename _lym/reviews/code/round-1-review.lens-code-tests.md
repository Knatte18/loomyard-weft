```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it:**

- `CheckPromptEngine` (own engine passes, hand-written passes, another engine refused, unset refused, refusal names both engines and "model setting"): `TestCheckPromptEngine_RefusesAPromptRenderedForAnotherEngine`, rows OwnEngine, HandWritten, OtherEngine, Unset.
- Claude `Prepare` runs the check first and writes nothing on refusal: claudeengine `TestPrepare_PromptEngine`, all four rows.
- Codex `Prepare` runs the check first and readies no home on refusal: codexengine `TestPrepare_PromptEngine`, all four rows.
- `EngineSet.EngineName` (empty choice gives the default, a named engine passes through): `TestEngineSetEngineName_EmptyChoiceTakesTheDefault`.
- `Runner.EngineName` and the `engineFor` refactor: `shuttleengines_test.go`, whose spec takes `runner.EngineName(ModelChoice{})`; existing runner tests reach `engineFor`.
- The scan (Prompt without an engine, Prompt with one, probe skipped, hand-written inside and outside shuttlecli, the real module): `TestPromptEngineRule`, its five rows and RealModule.
- `lym shuttle run` marks the spec hand-written for inline `--prompt` and for `--prompt-file`: `TestRunCmd_FocusFlag`, the HandWrittenPrompt assertion with the new PromptFile row.
- Orch spec pinned to Claude: `TestStart_NoStrandLaunchesAndSpawnsWatcher`, the claude/claude assertion.
- Orch remote-control check now looks up the Claude engine whatever the default is: none.
- `editdirective.Directive(dir, engine)`, reading each engine's copy, and the error for a missing or unshipped copy: `TestDirective_ReadsTheEnginesCopyWithoutBanner`, `TestDirective_EngineWithoutACopyNamesTheStencil`.
- The edit-rule guard exempts only the engine copies: `TestStencils_NoEditRuleRestatement`.
- Each producer sets PromptEngine to the name its namer resolves:
  - bouncer seed and judge: `bouncer_judge_test`
  - treadle circling and milestone, on codex: `judge_test`
  - sampler, plus the nil-Engines refusal row: `samplerengine/spec_test`
  - PR-Review: `prreview_session_test`
  - friction: `reflect_test`
  - mergeresolve: `mergeresolve_test`
  - dobby: `tidy_test`
  - arbiter: `prompt_test`
  - tackler: `tacklerprompt_test`
  - loom driver: `driverspec_test`
  - weaver batch and fixer, first rung only: `run_test`
  - webster master and recovery: `runlevel_test`, `recoverbatch_test`
  - webster verify fixer: `verifyfixer_test`
  - gatefixer threading to the shuttle spec: `TestSpawnOrAttach`
  - seat spec field: `TestSeatPrompts_ValuesPromptsAndSpecs`
- The rendered prompt carries the copy of the engine the launch resolves to, not a fixed one: pinned only for assistant seats, by seatengine `prompt_test.go:315-316`. No other producer has such a row.
- Every non-assistant seat now reads its engine's copy: none. `TestPrompts_RenderEachSeatsEngineBlocksForItsEngine` puts a chair on codex but asserts only its engine-block words, not the edit directive.
- loomcli gate fixer and gatecli card fixer set `gatefixer.Spec.PromptEngine`: none. `loomcli/gatefixer_test.go:249` and `gatecli/cardfixer_test.go:289` assert prompt text only, and the scan cannot see a `gatefixer.Spec` literal.
- Weaver's engine follows the rung each attempt runs on: only the first rung, on claude.
- `runnerTidyShuttle.EngineName` falls back to the choice's own engine when the runner build fails: none.
- The bouncer's `promptRenderError` keeps the degrade reason and cause it logs: no test asserts the reasons, but they were unasserted before the split, so this adds no new gap.

**Findings**

1. MEDIUM, design. **Location:** producers outside seatengine (bouncer, treadle, sampler, landingshed, friction, mergeresolve, dobby, arbiter, tackler, loom driver, weaver, webster, gatefixer) and seatengine's non-assistant seats. **Summary:** No test asserts that a producer's rendered prompt carries the edit-directive copy of its launch engine, which is acceptance criterion D3 ("Every producer's prompt for a Codex launch holds the Codex edit directive"). Every producer test fakes `EngineName` to "claude", or asserts only `PromptEngine`. A producer that rendered `editdirective.Directive` for a fixed engine while setting `PromptEngine` correctly would pass every test, and `Prepare` would not catch it either. Which copy gets picked is a fragment Go chooses, not static prose, so the exemption does not apply. **Evidence:** `internal/treadleengine/judge_test.go` (codex rows assert `spec.PromptEngine` only); `internal/seatengine/prompt_test.go:315-316` (assistant rows only); `prompt_test.go:367-399` (codex chair, no edit-directive assertion); `internal/samplerengine/spec_test.go:39`, where the namer returns `choice.Engine` for a claude choice.

2. LOW, design. **Location:** `internal/orchcli/remotecontrol.go:42-46` (`orchModelChoice` now pins `Engine: orchEngine`). **Summary:** `orchHasRemoteControl` now looks up the Claude engine whatever the default engine is, and no orchcli test drives that check under a Codex default. **Evidence:** a grep of `internal/orchcli/*_test.go` for `orchHasRemoteControl|orchModelChoice|codex` returned nothing.

3. LOW, design. **Location:** `internal/loomcli/gatefixer.go:272-292` and `internal/gatecli/detachfixer.go:270-284`. **Summary:** `PromptEngine` on the loom row fixer's and the gate card fixer's `gatefixer.Spec` is never asserted. The scan sees only `shuttleengine.Spec` literals, so dropping the field would pass every tier0 test and fail only at `Prepare`. **Evidence:** `internal/loomcli/gatefixer_test.go:249,254` and `internal/gatecli/cardfixer_test.go:289` assert prompt substrings only.

4. LOW, design. **Location:** `internal/weaverengine/run.go:447` and `internal/weaverengine/verify.go:295`, which take the engine from `deps.Ladder[rung]`. **Summary:** The behaviour the summary claims, "The weaver's engine follows the ladder rung each attempt runs on", is pinned only for the first rung on claude. No row escalates to a rung on another engine. **Evidence:** the `weaverengine/run_test.go` diff asserts `spec.PromptEngine != "claude"` for the first rung only.

5. LOW, design. **Location:** `internal/beamcli/dobbydeps.go:692-698`. **Summary:** The fallback in `runnerTidyShuttle.EngineName` (runner build failed → return `choice.Engine`) has no test. **Evidence:** of all `EngineName` implementations, only test files under other packages were found; none in beamcli.

Tool calls:
1. `git diff` of the `*_test.go` files, grepped for PromptEngine and EngineName assertions: the manifest held only a subset of the test diffs.
2. `git diff` of the treadle, weaver, webster, bouncer, landingshed, loomcli and weavercli test files: their assertions were not in context.
3. A grep of the orchcli, samplerengine, gatecli and loomcli fixer, seatengine and burlerengine tests and the bouncer degrade reasons: to check coverage of those behaviours.
4. A grep of the fixer-spec assertions and a read of `seatengine/prompt_test.go:360-400`: to confirm whether PromptEngine or the codex directive is asserted there.
