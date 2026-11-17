# Lens review

Every lens returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.

## coverage

- coverage F1 (MEDIUM), manifest built from Prepare's inputs, not read back from the launch: fixed differently. The proposal was no specific change; cards 3 and 4 now keep a candidate item only when the written artifact (settings.json, appended prompt, launch line, marked heddle file) holds its text or command, and the overview's first Shared Decision says so.
- coverage F2 (MEDIUM), Claude and Codex items of most channels unpinned: fixed as proposed. Card 3's and card 4's `TestPrepare_WritesTheManifest` gain rows for every channel: instruction sources, skill listing, plugin context, prompt engine and hand-written, subagent passage, standing passage, batching nudge, and the pointer branch.
- coverage F3 (MEDIUM), Claude's repository root and working directory unstated: fixed as proposed. Card 3 resolves the anchor with `filepath.EvalSymlinks` and takes the nearest `.git`-holding ancestor as the root.
- coverage F4 (MEDIUM), fork deny keyed to claude without licence, and the skip untested: fixed differently. The proposal was no specific change. The fork deny is now compared (`webster-fork-deny`, empty `Engine`); a seat Codex cannot start goes through `shuttleparity.Unmet`, which has `TestCompare` rows; the overview decision is replaced.
- coverage F5 (MEDIUM), D11 written to `codexengine` and not `codex-engine-trial`, plus the single-session contradiction and the dropped replay clause: fixed differently. No entry `codex-engine-trial` exists, and P1's body records its merge into `codexengine`, so the write stays there; the parent recorded the call as an `Added after Discussion` entry (2026-11-17), which the overview cites. Card 10 no longer contradicts `## The one run` and now states the redesigned replay route.
- coverage F6 (MEDIUM), the verb's seats built from stand-ins in place of the current worktree: fixed as proposed. Card 6's `Build` loads each builder's config and language skills from the worktree's layers, with stand-ins only for a run's own inputs.
- coverage F7 (MEDIUM), the verb's ok and err exits never run: fixed as proposed. Card 8's `TestManifestCmd` drives a hand-built `shuttleCLI` through both exits.
- coverage F8 (MEDIUM), the edit-directive not-delivered branch was unreached: fixed as proposed. `TestCompare` gains `PromptItems` rows with and without the copy.
- coverage F9 (LOW), the orch skip had no row: fixed as proposed. Card 7's scan table gains `orchcli` and `orchengine` rows.
- coverage F10 (LOW), copy failure and unreadable record had no row: fixed as proposed. Card 2 adds a row with a file at `manifests`, and states that an unreadable record is logged at Warn.
- coverage F11 (LOW), nil-seam fallbacks untested: disputed, every fix worse. Reaching them runs `claudeengine.RunPluginHook`'s `sh -c` (`internal/shuttleengine/claudeengine/pluginhook.go:189`) and `runSandboxProbe`'s `codex` (`internal/shuttleengine/codexengine/sandbox.go`), both of which `PATTERN-test-speed` bars from tier0. A tier1 test for a nil default adds a spawning test for a one-line wiring; card 5 states the bound.
- coverage F12 (LOW), `Passage.Engine` double-aimed entry and the `StandingPassages` error branch: fixed as proposed. Card 1 adds a `TestRenderer_EntryKeys` row and `TestStandingPassages`.
- coverage F13 (LOW), DenyPublish and DenyPython rows: fixed as proposed in cards 3 and 4.
- coverage F14 (LOW), `PrepareEach`'s absent-record branch: fixed as proposed. A `TestReport` row covers it.
- coverage F15 (LOW), Codex told the operator to fix hooks that were only listed unrun: fixed as proposed. Card 2 adds `ErrHookUnrun`, card 4 keeps it out of the warning (with a row), and card 6's `UnrunHooks` returns it.

## row-validity

- row-validity F1 (MEDIUM), card 5's probe and hook were unreachable on the named fixture: fixed as proposed. The test rewrites the layer to `auto` and writes the plugin itself. Folding it into `TestNew_CodexLaunchCarriesTheClaudeUserSetup` was not done: that test pins `New`'s production wiring, this one `NewWithSeams`'s.
- row-validity F2 (MEDIUM), the verb test could not pass `PersistentPreRunE` at tier0: fixed as proposed. The verb's environment is now a `shuttleCLI` field, the test builds the receiver by hand, and the no-argument row moves into `TestCommand_PositionalArgValidation`.
- row-validity F3 (MEDIUM), the verb's non-zero exit was unreached: fixed as proposed, through the same injectable field (`TestManifestCmd`).
- row-validity F4 (MEDIUM), `PartsPlanTable` and `DescribeTable` could not build on stand-ins: fixed as proposed. Card 6's `Build` writes the parts file and decision record under `env.Dir` and fills the branches.
- row-validity F5 (MEDIUM), card 3 had no native-item rows and no config directory: fixed as proposed. Rows set `claudeDir` to a fixture holding a global CLAUDE.md, a plugin and a skill.
- row-validity F6 (MEDIUM), card 4's skill-load row could not fail, other channels were unasserted, and `fakeUserSetupSource` was unlisted: fixed as proposed.
- row-validity F7 (LOW), `fakeEngine.manifest` duplicates `PrepareHook`: fixed as proposed. Card 2 uses `PrepareHook`; `fakeEngine` moves to Uses.
- row-validity F8 (LOW), the sweep test is no table: fixed as proposed. Card 2 now seeds the directory and asserts it survives.
- row-validity F9 (LOW), `HoldDeferral` comparison: fixed as proposed. Card 6 nils both specs' field after asserting the lead's was set.

## forced-edits

- forced-edits F1 (BLOCKING), the constructor re-sign broke unlisted code: fixed as proposed. Card 5 lists `build`, `keepWarmWith`, `KeepWarm`, `SessionNamers`, `RecordReaders`, `ToolBoundaryRenderer`, `BatchingNudger`, `EditRecorder`, `TestKeepWarm` and `TestNew` under Edit.
- forced-edits F2 (MEDIUM), `TestRender` calls `Passages`: fixed as proposed in card 1.
- forced-edits F3 (MEDIUM), the Codex refusal asserts `heddle passage 1`: fixed as proposed. Card 1 states the expectation change.
- forced-edits F4 (LOW), `docs/overview.md` lists shuttle verbs and packages: fixed as proposed. Card 8 extends `//docs/overview.md`.
- forced-edits F5 (LOW), the cli.go header lies outside `Command`'s span: fixed differently. Listing `internal/shuttlecli/cli.go` beside `internal/shuttlecli#Command` is refused by `redundant-file-target`, so card 8 has `Command`'s own doc comment name the verb.
- forced-edits F6 (LOW), the seam background does not mention `NewWithSeams`: fixed as proposed. Card 5 extends `//pattern/PATTERN-shuttle-provider-seam.md`.
- forced-edits F7 (LOW), `promptEngineViolations` is test code of another package: fixed as proposed. Card 7 drops it from Uses and says the walk is its own.

## guard-reach

- guard-reach F1 (BLOCKING), the Table scan fired on empty literals, helper holders and MultiLLM: fixed as proposed. Card 7 skips empty `seatengine.Table{}`, card 6's entries carry `Holders`, and the listed set names `internal/shedrecipe#multiLLMEntry`.
- guard-reach F2 (BLOCKING), `seatSpec`'s literal was unlisted: fixed as proposed. Card 7's listed set holds `internal/seatengine#seatSpec`.
- guard-reach F3 (MEDIUM), heddle file items were read off the reader, not the rendered file: fixed as proposed. Cards 3 and 4 require the marked file to hold the entry, with an unmarked-file row; card 6's fixture writes both marked files.
- guard-reach F4 (MEDIUM), the verb's stand-ins landed outside the temporary directory, and one builder error blocked every seat: fixed as proposed. `Report` makes the directory first and sets `env.Dir`; `Seats` returns per-builder errors, and `LookupSeat` refuses only a failed builder's seats.
- guard-reach F5 (LOW), card 7's forks skip was unreachable: fixed as proposed. Producers carry `ForkSubagents` and `Requires`, and Merriam's spec reaches the skip.
- guard-reach F6 (LOW), card 5's probe was unreachable: merged with row-validity F1, fixed as proposed.
- guard-reach F7 (LOW), copies outlived failed launches: fixed as proposed. Card 2 copies after `saveRunState`.

## way-forward

- way-forward F1 (MEDIUM), a builder failure blocked the verb for every seat with no next step: merged with guard-reach F4, fixed as proposed.
- way-forward F2 (MEDIUM), the missing-item exit named no next step: fixed as proposed. Card 8's error names the codexengine fix and holds the bench.
- way-forward F3 (LOW), the parity and scan failures named no next step: fixed as proposed in cards 6 and 7.
- way-forward F4 (LOW), `PrepareEach`'s absent record named no next step: fixed as proposed. The error names `shuttleengine.WriteManifest`.
- way-forward F5 (LOW), a failed manifest write named no next step: disputed, premise false. It is a propagated raw I/O error, which `refusal-spec.md` exempts as transient, and the lens itself confirms the launch is retryable (`internal/shuttleengine/run.go:649-652`).
- way-forward F6 (LOW), the verb had no rule for a seat Codex cannot start: fixed as proposed. `Report` returns `Unmet`, and the verb refuses naming the requirement.
