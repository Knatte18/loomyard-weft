## Fixer-report, code review round 1

The gate over every touched package ended green: tier0, no failures.

## Changes

- F1 is `decision`-class and went to the parent; see Rulings.
- F2: `internal/editdirective/editdirective_test.go` now sits in `package editdirective_test` and qualifies `Directive` and `EngineBlock`.
  `testmain_test.go` stays in `package editdirective`.
  `stencilkit` deleted its `editDirectiveBlock` constant and uses `editdirective.EngineBlock`.
  `stencilkit` gained `RequireEditDirectiveOf`, which F4 uses.
- F3: `TidyShuttle.EngineName` returns `(string, error)`, and `spawnTidy` returns that error, wrapped, before it renders the prompt.
  `runnerTidyShuttle.EngineName` returns the runner build's error, with its doc comment corrected.
  The tidy fake gained `engineErr`, and a subtest asserts that a failing `EngineName` starts nothing and records no tidy.
- F4: each producer test the review names now asserts the rendered prompt carries the launch engine's edit directive and not the other engine's, through `stencilkit.RequireEditDirectiveOf`.
  Rows on codex were added or extended in these tests:
  - treadleengine judge, milestone and targeting;
  - samplerengine, landingshed and frictionengine;
  - mergeresolve, dobbyengine tidy, shedarbiter and sheddriver tackler;
  - loomcli driver, weaverengine and websterengine (master and recovery);
  - gatefixer and shedadapters bouncer.
- F5: `driverPrompt` is now `RenderDriverPrompt`, with its callers, doc comments, the three driver stencil banners and the integration test comment updated.
  The design-decision bullet in `_lym/landing/summary.md` is corrected. I did not commit that file, since it lives under `_lym`.
- F6: the arbiter, tackler, board-tidy and driver banners name the edit directive of the engine the prompt launches on.
- F7: a `remotecontrol_test.go` row runs the orch under an engine set whose default is codex and asserts the launch still passes and records remote control.
- F8: `loomcli` `TestFixerSpec` and `gatecli` `TestRunWithFixer` assert the built `gatefixer.Spec`'s `PromptEngine`.
- F9: the weaver failure-climb rows give rung 1 another engine and assert each attempt's `PromptEngine`.
  The fixer-ladder test asserts each fixer's `PromptEngine` per rung.
- F10: `//testtiming:keep` marks went on `TestProducerSpecs_SkillsAndParentDirective`, `TestBouncerEntry_EnvJudgeFallback`, both `TestPrepare_AppendsStandingPassages`, `TestPrepare_LogsResolvedPromptCacheTTL`, `TestPrepare_AdvisorReachesSettings`, `TestPrepare_CreelMailHookFollowsTheBoardDirectory`, `TestPrepare_EffortFromAgentsConfig` and `TestPrepare_NoAgentsConfigWritesNoSubagentRule`.
  I compared `EffortFromAgentsConfig`, `NoAgentsConfigWritesNoSubagentRule` and `CreelMailHookFollowsTheBoardDirectory` against their covering tests, and narrowed each keep reason to what the covering tests miss.
  I judged the claudeengine `AppendsStandingPassages`, `AdvisorReachesSettings` and `LogsResolvedPromptCacheTTL` marks from a grep of their covering tests for the asserted strings.
  I did not read the covering tests of the codexengine `AppendsStandingPassages` in full.

A gate finding on the round's comments followed: the doc comment of `RequireEditDirectiveOf` wrapped one sentence over two lines.
I rewrote it as two sentences, one per line.

Each finding is one commit, `<target>: fix <id> — ...`.
Commits for overlapping files went to the first finding that touched them: `tidy_test.go` with F3, `driverprompt_test.go` and `shed-template-driver.md` with F5.
The self-check fix is its own commit.

### Fixed differently

- F4, the seatengine part: I did not add the assertion to `TestPrompts_RenderEachSeatsEngineBlocksForItsEngine`.
  The assertion failed, because the seat chair and advisor stencils there carry no edit directive.
  The seat table's assistants carry it, and `internal/seatengine/prompt_test.go:315-316` already pins each assistant's engine copy.

## Disputed

- F4, seatengine part: ground `premise false`.
  The review expects the codex chair's prompt to carry an edit directive copy.
  The gate failed on that assertion (`prompt lacks the codex edit directive`), since the chair carries none; the engined chair in that test holds only the engine block "test-words".

## Deferred

Nothing.

## Rulings

- F1: the parent recorded that the orch's Claude pin stays a code literal, with no orch config key.
  The code already holds that, so I changed nothing.
  I did not tell the parent, since the ruling is neither mistaken nor in conflict with the code.

## Self-check

- fix-consistency, LOW: `loom-template-discussion-advisor.md` banner still called the edit directive shared. Fixed as proposed.
- fix-code-correctness, LOW: the keep mark on `TestPrepare_EffortFromAgentsConfig` overclaimed. Fixed as proposed.
- fix-proposal, LOW: the same mark overclaimed. Duplicate, merged with the finding above.
- fix-proposal, LOW: the keep marks on the remaining claudeengine tests were unverified. Fixed with my own change:
  I checked the covering tests by grep and narrowed the `NoAgentsConfig` and `CreelMailHook` reasons, and kept the other claudeengine marks, since no covering test asserts what they pin.
- fix-affected-set: no findings.
