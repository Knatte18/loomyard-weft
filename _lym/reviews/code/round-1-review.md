---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    origin: lens:code-affected-set
    location: internal/orchcli/remotecontrol.go:42
    summary: The new const orchEngine = claude is a caller naming an engine, which PATTERN-code-conventions forbids, though D4 asks for the orch to be pinned to Claude
    fix: Delete the orchEngine literal in internal/orchcli/remotecontrol.go. Read the orch's engine from the orch config instead, as a new engine key loaded through configengine whose shipped template default is claude. orchModelChoice sets ModelChoice.Engine from that key, and orchSpec in internal/orchcli/start.go sets PromptEngine from the same value. Extend the orch config's template and loader and the TestStart_NoStrandLaunchesAndSpawnsWatcher assertion to match. If the run's parent instead rules that the pin stays a code literal as D4 words it, leave the code in place, defer this finding to the parent, and have the parent record that exception with lym loom decision add.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/testkit/stencilkit/stencilkit.go:40
    summary: stencilkit declares a second editDirectiveBlock = edit-directive constant, repeating editdirective.EngineBlock against the one-named-constant rule
    fix: Change internal/editdirective/editdirective_test.go from package editdirective to package editdirective_test, qualifying Directive and EngineBlock as editdirective.Directive and editdirective.EngineBlock. testmain_test.go stays in package editdirective. Then delete the editDirectiveBlock constant and its comment in internal/testkit/stencilkit/stencilkit.go, import internal/editdirective there, and use editdirective.EngineBlock in SeedEditDirectiveCopies.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/beamcli/dobbydeps.go:692
    summary: When the shuttle runner cannot be built, runnerTidyShuttle.EngineName returns the choice's own engine, often empty, so TidyPrompt fails on a missing engine/edit-directive stencil and the runner build error is never reported
    fix: In internal/dobbyengine/tidy.go, replace the shuttleengine.EngineNamer embedded in TidyShuttle with a method EngineName(choice shuttleengine.ModelChoice) (string, error), and have spawnTidy return that error, wrapped as resolving the board-tidy agent's engine, before it renders TidyPrompt. In internal/beamcli/dobbydeps.go, have runnerTidyShuttle.EngineName return ("", s.err) when the runner build failed and (s.runner.EngineName(choice), nil) otherwise, and correct its doc comment to say so. Update the TidyShuttle fakes in internal/dobbyengine/tidy_test.go to the new signature, and add a row to an existing dobbyengine test whose fake EngineName fails, asserting that spawnTidy returns that error and starts nothing.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/treadleengine/judge_test.go
    summary: No test asserts that a producer's rendered prompt carries the edit-directive copy of the engine its launch resolves to; tests check only PromptEngine, so a producer rendering a fixed engine's copy would pass
    fix: In each producer test that already asserts PromptEngine (treadleengine judge_test and targeting, samplerengine spec_test, landingshed prreview_session_test, frictionengine reflect_test, mergeresolve mergeresolve_test, dobbyengine tidy_test, shedarbiter prompt_test, sheddriver tacklerprompt_test, loomcli driverprompt_test, weaverengine render_test, websterengine template_test, gatefixer prompt_test, shedadapters bouncer_judge_test), add or extend one row whose namer resolves to codex. Assert that the rendered prompt contains the string editdirective.Directive(stencilsDir, codex) returns, and does not contain the claude copy's string. Comparing against Directive's output pins Go's choice of copy rather than the stencil's prose. In internal/seatengine/prompt_test.go, extend TestPrompts_RenderEachSeatsEngineBlocksForItsEngine with the same assertion for its codex chair.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/loomcli/driverprompt.go:58
    summary: The loom driver's prompt render function driverPrompt stays unexported, so launch-manifest-parity's producer table in internal/shuttleparity cannot call it, against the Exports clause for every producer outside seatengine
    fix: Rename driverPrompt in internal/loomcli/driverprompt.go to the exported RenderDriverPrompt, keeping its signature and engine parameter, and update its doc comment's opening and its callers in internal/loomcli/start.go and internal/loomcli/driverprompt_test.go. Correct the design-decision bullet in _lym/landing/summary.md that says loomcli's driver prompt stays unexported.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/loom/loom-template-arbiter.md:17
    summary: Four opening-stencil banners still call the edit_directive marker the shared edit directive, though no shared copy exists any more
    fix: In the banner comments of contracts/stencils/loom/loom-template-arbiter.md:17, contracts/stencils/shed/shed-template-tackler.md:16, contracts/stencils/beam/beam-template-board-tidy.md:6 and contracts/stencils/shed/shed-template-driver.md:13, replace the shared edit directive with the edit directive of the engine the prompt launches on, keeping the rest of each line.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/remotecontrol.go:46
    summary: orchHasRemoteControl now looks up the orch's pinned engine whatever shuttle's default engine is, and no test drives that check under a Codex default
    fix: In internal/orchcli/remotecontrol_test.go, add a row to the existing remote-control test with a Codex default engine in the engine set, asserting that orchHasRemoteControl still answers from the orch's pinned engine.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/gatefixer.go:272
    summary: PromptEngine on the gatefixer.Spec the loom row fixer and the gate card fixer build is never asserted, and the scan does not see gatefixer.Spec literals
    fix: In internal/loomcli/gatefixer_test.go and internal/gatecli/cardfixer_test.go, add an assertion to the existing fixer-spec tests that the built gatefixer.Spec's PromptEngine equals the name the test's engine namer returns for the fixer's model choice.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/weaverengine/run.go:447
    summary: That the weaver's prompt engine follows each attempt's ladder rung is pinned only for the first rung on claude
    fix: In internal/weaverengine/run_test.go, add a row whose ladder escalates to a rung naming another engine, and assert that the escalated attempt's spec and its fixer's gatefixer.Spec carry that rung's engine as PromptEngine.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: internal/loomengine/discussion_test.go
    summary: Tests the redundancy report lists as contained, and that the diff changed, pin what their covering tests never assert, yet carry no testtiming keep mark
    fix: Add a //testtiming:keep line directly above each func line. For TestProducerSpecs_SkillsAndParentDirective in internal/loomengine/discussion_test.go, the reason is that it pins the told parent, the no-parent variant and the operator-ban line, which TestDiscussionTable and TestPlanTable do not assert. For TestBouncerEntry_EnvJudgeFallback in internal/shedrecipe/entries_bouncer_test.go, it pins each spawn's model triple and its role and the attach probe's role, which its covering tests do not assert. For TestPrepare_LogsResolvedPromptCacheTTL in internal/shuttleengine/claudeengine/prepare_test.go, it pins the role and TTL in the log line. For TestPrepare_AppendsStandingPassages in the same file, it pins the passages appended to append-system-prompt.txt and the reader-error log. Then compare each other contained test in the report against its listed covering tests the same way, and either mark it keep with what it pins or fold it into a covering test. Those are claudeengine TestPrepare_AdvisorReachesSettings, TestPrepare_CreelMailHookFollowsTheBoardDirectory, TestPrepare_EffortFromAgentsConfig and TestPrepare_NoAgentsConfigWritesNoSubagentRule, and codexengine TestPrepare_AppendsStandingPassages.
---

### [BLOCKING] The orch's pinned engine is a hard-coded engine literal

**Location:** `internal/orchcli/remotecontrol.go:42`

**Issue:** The diff adds `const orchEngine = "claude"`, and `orchModelChoice` and `orchSpec` take the orch's engine and prompt engine from it.
`PATTERN-code-conventions` § No hard-coded choice says an engine "is chosen by config and reached through shuttle's provider seam; no caller names an engine", and that a change touching an existing literal "never adds another".
`orchcli` already named claude in its remote-control refusal, and this change adds a second literal.
D4 asks for "The orch spec names the Claude engine and its prompt engine explicitly".
The listed constraint and the answer key conflict here, and the prompt's rule says the constraint wins.
A config key whose shipped default is claude meets both: the spec still names Claude explicitly, and a Codex default engine still never takes the orch role.
The finding is filed as `decision` because the parent may instead record the literal as an exception.

### [BLOCKING] stencilkit repeats the edit-directive block name

**Location:** `internal/testkit/stencilkit/stencilkit.go:40`

**Issue:** `stencilkit` declares `editDirectiveBlock = "edit-directive"`, a second copy of the name `editdirective.EngineBlock` owns at `internal/editdirective/editdirective.go:18`.
`PATTERN-code-conventions` § Named constants requires one named constant, declared once and referenced everywhere.
The comment gives an import cycle as the reason: `editdirective_test.go` is an internal test, `package editdirective`, that imports stencilkit.
That test uses only exported names (`Directive`, `EngineBlock`), so moving it to `package editdirective_test` removes the cycle.
The fork filed this as LOW; it is raised to BLOCKING because the constraints section makes every violation of a listed constraint BLOCKING.

### [MEDIUM] A failed runner build is reported as a missing stencil

**Location:** `internal/beamcli/dobbydeps.go:692`

**Issue:** On `s.err`, `runnerTidyShuttle.EngineName` returns `choice.Engine`, and the shipped board-tidy model usually names no engine.
`spawnTidy` (`internal/dobbyengine/tidy.go:91-94`) renders `TidyPrompt` before `Start`, so `editdirective.Directive` fails on `engine/edit-directive` and returns that error.
The runner build error is never reported.
The method's comment says "the start that follows fails with the build's error", which is false on this path.
No test reaches this fallback; the code-tests lens's item 5 is folded into this finding.

### [MEDIUM] No test pins that a prompt carries its launch engine's edit directive

**Location:** `internal/treadleengine/judge_test.go`

**Issue:** D3's acceptance criterion is "Every producer's prompt for a Codex launch holds the Codex edit directive".
The producer tests assert `spec.PromptEngine` against the namer, but none asserts which edit-directive copy the rendered prompt holds.
The one exception is seatengine's assistant rows (`internal/seatengine/prompt_test.go:315-316`).
A producer that passed a fixed engine to `editdirective.Directive` while setting `PromptEngine` correctly would pass every test and every `Prepare`.
The copy is a fragment Go chooses, not static stencil prose, so the prose exemption does not apply.

### [MEDIUM] The loom driver's render function is not exported

**Location:** `internal/loomcli/driverprompt.go:58`

**Issue:** The board entry's Exports clause asks for "for every producer outside seatengine, one prompt render function that takes the engine name and needs no session", and says "launch-manifest-parity's producer table calls these".
`lym board get` shows that task's parity test lives in a new `internal/shuttleparity` package, which cannot call the unexported `driverPrompt`.
The parenthetical list names where a launcher must be split, not which producers get an exported renderer.
So the summary's reason, that the record's export list does not name CLI packages, does not hold for the driver.
The gate card fixer is no gap: it renders through the exported `gatefixer.RenderPrompt`.

### [LOW] Opening-stencil banners still describe a shared edit directive

**Location:** `contracts/stencils/loom/loom-template-arbiter.md:17`

**Issue:** The arbiter, tackler, board-tidy and driver templates' banner comments say `{{.edit_directive}}` is "the shared edit directive rendered by internal/editdirective".
The diff removes the shared stencil, and `internal/editdirective/doc.go` and `pattern/PATTERN-stencil-ownership.md` now state that no shared copy exists.

### [LOW] The orch remote-control check has no row under a Codex default

**Location:** `internal/orchcli/remotecontrol.go:46`

**Issue:** `orchHasRemoteControl` looks up `orchModelChoice().Engine`, which the diff changes from the default engine to the pinned one.
No orchcli test runs that lookup with a Codex default engine.
The summary claims "the same choice now drives the orch's remote-control check", and nothing pins that claim.

### [LOW] The fixer specs' PromptEngine is unasserted

**Location:** `internal/loomcli/gatefixer.go:272`

**Issue:** `loomcli`'s `fixerSpec` and `gatecli`'s `cardFixerSpec` set `gatefixer.Spec.PromptEngine`.
`TestPromptEngineRule` sees only `shuttleengine.Spec` literals, and the fixer tests assert prompt text only.
If the field were dropped, the failure would show only at `Prepare` in a live launch.

### [LOW] Weaver's per-rung engine is pinned for the first rung only

**Location:** `internal/weaverengine/run.go:447`

**Issue:** `runBatch` and `fixerSpec` take the engine from `deps.Ladder[rung]` and `deps.Ladder[fixer.Rung]`.
The run test asserts `PromptEngine` for the first rung only, so code reading `deps.Ladder[0]` would pass.

### [LOW] Contained tests the diff touched lack keep marks

**Location:** `internal/loomengine/discussion_test.go`

**Issue:** The redundancy report lists these tests as contained.
Each was checked against its covering tests' assertions.
`TestDiscussionTable` and `TestPlanTable` assert config role and skills, but not the parent name, the no-parent variant or the operator-ban line.
`TestBouncerEntry_ApproveSeam`, `TestBouncerEntry_ConstructionFailures` and `TestBurlerHalfEntries_ColorSegment` assert neither model triples nor spawn roles.
The claudeengine covering tests assert neither the TTL log line nor the appended passages.
Each therefore pins something coverage cannot see, and gets a keep mark rather than a fold.
The remaining contained claudeengine and codexengine tests were not compared assertion by assertion in this round, so the fix asks the fixer to make that comparison.

## Rejected

- The `"claude"` default in `internal/testkit/shedfake/shedfake.go:79` and `internal/testkit/stencilkit/stencilkit.go:46`: test support picks an engine that every seeded stencil set ships a copy for, and the no-hard-coded-choice rule governs lym's own code paths.
- The refusal not naming a specific config key as the "producer's model setting": `CheckPromptEngine` names the role and both engines, and tells the reader to set the producer's model setting. The spec carries no setting key it could name, and D4's wording is met.
- The code-tests lens's item 5 (`runnerTidyShuttle.EngineName` fallback untested): folded into F3, whose fix adds the test.
