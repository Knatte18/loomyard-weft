```yaml
lens: affected-set
outcome: reported
```

**Lens affected-set: for each change a card makes, what reads or asserts it, and which cards cover that set**

**Per change, the set found and the cards covering it**

- **Card 5 – prompt.md holds the prompt alone; standing items move to `developer_instructions`.**
  - Asserters that read prompt.md:
    - `TestPrepare_Launch` (prepare_test.go:134)
    - `TestPrepare_SandboxMode` (:200–221)
    - `TestPrepare_ExpandsHomeInWritableRoots` (:269)
    - `TestPrepare_InstallsGuardsFromItsInputs` (:347)
    - `TestPrepare_AppendsStandingPassages` (:362ff)
  - `TestPrepare_RefusesBeforeWriting` (:628) asserts only that prompt.md is absent, so it is unaffected.
  - Docs: `docs/overview.md:711` ("the creel trust sentence in its prompt file"), `OpenLaunch`'s doc comment, the `guards.go` header and `codexengine/doc.go`.
  - `promptText` has one caller, `Prepare`.
  - Covered in full by card 5.
- **Cards 5, 6 and 10 – new `-c` settings on the Prepare launch line.**
  - Builders of the full want line: `TestBuildLaunchCmd` (command_test.go:32–50) and the four Prepare tests above (prepare_test.go:117, 229, 270, 335).
  - Lines that stay as they are:
    - `codexCommand` in `TestBuildLaunchCmd` (command_test.go:57) and `TestOpenLaunch` (open_test.go:133): their zero-valued new fields add nothing.
    - `TestPrepare_SkillsLoadFirst` (skillload_test.go:298): it checks with Contains, and the new settings sit before the hook settings.
  - Cards 5, 6 and 10 each list these tests.
  - Missed: the full-line tests that cards 6 and 9 create. See F1.
- **Card 2 – the `agents-prompt` row and `Show`'s refusal text.**
  - Only `render.go:341` holds the refusal string, and no test asserts it outside `TestRender`.
  - The heddlecli list and help output come from `fileTargets()`, which uses `TargetFiles` (verbs.go:52–69). That returns nil for a prompt target, so the `file_targets` assertion (cli_integration_test.go:267–277) and the set-help lines are unaffected.
  - The `TargetNames()` "use one of" messages (store.go:372, 376; render.go:338) gain the name, and no test pins the full list.
  - `HEDDLE.md`'s intro: `TestRenderOverview` checks with Contains (overview_test.go:55–75), so it is unaffected.
  - The doc comments of `Show` (render.go:334) and `reachesFileTarget` are covered by card 2's Edit targets.
  - Missed: the `Target` type's doc. See F2.
- **Card 4 – codex-file carries both AGENTS.md and CLAUDE.md.**
  - Asserters:
    - `TestRender` `CodexFileOpensWithRootAgents` (covered).
    - heddlecli cli_integration_test.go:776–780: Contains on the AGENTS text, still true.
    - seam_integration_test.go:88–96: prefix and suffix checks; the Marker and AGENTS text still open the file and the entries still close it.
  - Docs: `heddleengine/doc.go:13` (card 4 Edits `doc.go`) and render.go:266, `fileText`'s doc (card 4 Edits `fileText`).
  - `RenderRequest` builders: `requestOf` and `heddlecli` `renderInto` (verbs.go:398, 505 call it).
  - Covered.
- **Cards 6 and 9 – new codex.yaml keys.**
  - Pins: `TestLoadConfig_Template` (covered by both cards) and the `ConfigSpec` classification (covered).
  - No other test names `max_subagent_threads` or `plugin_hook_timeout`.
  - Covered.
- **Card 9 – `NewFromConfig` and `readyHome` re-signed.**
  - Callers: the shuttleengines registry (shuttleengines.go:82), `newEngine` (fixture_test.go:149), and `Prepare` and `OpenLaunch`.
  - `New` (new_test.go:8) builds the struct directly.
  - Covered.
- **Card 3 – reconcile renders file targets.**
  - `hubreconcile/doc.go`, `fabricengine/doc.go` and `PATTERN-heddle-instructions.md` are listed.
  - Covered for the files I read.

**Findings**

- **F1 — MEDIUM, consistency, card 10 (`10-codex-instruction-files.md`).**
  - Summary: card 10 adds `-c project_doc_max_bytes=&lt;n&gt;` to every Prepare launch line. It updates only the four full-line Prepare tests and `TestBuildLaunchCmd`, and misses `TestPrepare_SubagentTerms`, which card 6 creates with full want lines.
  - Evidence:
    - card 6, `06-codex-subagent-terms.md:58–59`: "Each row builds its want line through `developerInstructions` and `tomlString`".
    - card 10, `10-codex-instruction-files.md:10–18` and `:66`: the Edit list names only `TestBuildLaunchCmd` and the four tests, which get `projectDocMaxBytes: defaultProjectDocMaxBytes`.
  - Effect: every row of `TestPrepare_SubagentTerms` fails at card 10's gate. The same method gap applies to `TestPrepare_UserSetup` (card 9) if its rows compare the whole line.
  - Fix: list `plan:internal/shuttleengine/codexengine#TestPrepare_SubagentTerms`, and `TestPrepare_UserSetup` if it compares the whole line, under card 10's Edit. Their want lines then carry the doc cap.
- **F2 — LOW, consistency, card 2 (`02-agents-prompt-target.md`).**
  - Summary: the `Target` type's doc says `RendersInto` names "the file targets whose files carry the entries of a shared file target". Card 2 adds a shared prompt target whose `RendersInto` names prompt targets, which makes that doc false, and card 2 does not list `internal/heddleengine#Target`.
  - Evidence: `internal/heddleengine/target.go:46–47`. Card 2's Edit list, `02-agents-prompt-target.md:6–16`, has `targetTable` but not `Target`.

**Tool calls**

1. grep of the codexengine tests for prompt.md, CreelTrustSentence and launch-line builders: the manifest lacked which tests assert prompt.md and the full lines.
2. grep across the repo for the `Show` refusal string, AGENTS.override.md asserters and docs that name the Codex prompt file: the asserters outside the pre-read were unknown.
3. Per-file grep of the heddle and heddlecli tests that touch AGENTS.override.md, plus the `fileText` doc lines: to judge whether the codex-file content change breaks them.
4. Read of the heddlecli list/help assertions, `overviewIntro` and the overview_test style: to see whether the agents-prompt row or the intro line breaks a pinned output.
5. Read of the heddlecli `fileTargets` and help generator, docs naming targets, and codex template key pins: to judge the new target and keys' reach.
6. grep for other Codex `Prepare` callers in tests, hookrun and guards_run line asserters, and pattern and docs mentions of the creel trust sentence: to close card 5's set.
