---
verdict: APPROVED
---

No findings.

The plan was judged against `_lyx/discussion/decision-record.md` alone; there is no rework generation (no `_lyx/loom/rework/` directory).

## Focus question 1 — test budget

Settled: not a defect.
The budget counts top-level `Test…` functions by the rule in `cmd/lyx/testbudget_test.go` (every build tag, `TestMain` excluded).
Counted against the tree, `internal/shuttleengine` holds 140 against a row of 140, `internal/shuttleengine/claudeengine` holds 89 against 89, `internal/configreg` holds 6 against 6, and `internal/webstercli` holds 81 against 81, so each row is exactly full.
- Card 1 adds one test function (`TestLoadConfig_PromptCacheTTLKeys`); its `**Edit:**` lists `cmd/lyx/testdata/test-budget.yaml` and its `**Intent:**` raises the `internal/shuttleengine` row 140 to 141.
- Card 3 adds one test function (`TestPrepare_LogsResolvedPromptCacheTTL`); its `**Edit:**` lists the same file and its `**Intent:**` raises the `internal/shuttleengine/claudeengine` row 89 to 90.
  Every other card 3 test change extends an existing function (`TestBuildLaunchCmd`, `TestBuildResumeCmd`, `TestBuildResumeCmd_NoticePosix`, `TestPrepare_RejectedBeforeArtifacts`, `TestPrepare_ThreadsSpecIntoLaunchCmds`) or edits a fixture, and the `internal/webstercli` smoke-test edit adds no function, so its row stays.
- Card 2 adds a row to `TestLookup`, no function, so the `internal/configreg` row stays.

## Focus question 2 — vocabulary containment

Settled: not a defect.
The `shuttleengine` files the plan edits are `Config`, `LoadConfig`, `template.yaml`, `doc.go` and `config_test.go`.
None names `CLAUDE_CODE_PROMPT_CACHE_TTL`, and none names the accepted values beyond the template defaults `5m`, `1h` for `driver` and `webster`.
- The `Config` field comments say the values are Claude Code's and validated by the engine.
- The `template.yaml` comments say the values are validated when a run starts and that the refusal names the accepted ones, without listing them.
- The `doc.go` sentence says the keys pass through uninterpreted.
- The `1h` and `5m` literals in the `config_test.go` rows are template-default values or fixtures for them.

The env var name, the accepted set, the validation and the log line are all confined to `claudeengine`.

## Other checks that came back clean

- Every Decision (D1 to D6) and Constraint is carried: the open map, `LoadConfig` passing it, and the `configreg` entry in cards 1 and 2; resolution, validation, the log line, both pane lines, the docs and the fixtures in card 3.
  D6 and the auto-mode assumptions need no card.
- Every fixture that hands `Prepare` a `Config` in the tree is covered by card 3's fixture sweep, including `TestPrepare_AppendSystemPromptMatchesDenyNotice`, `TestPrepare_ResumeSessionID` and `TestPrepare_SkillsDeferPromptPointer` (`Config{}` or deny-only literals in `prepare_test.go`), `envfile_integration_test.go`, and the `llm`-tagged `realForkSettingsPath` in `internal/webstercli/smoke_test.go`.
  The other packages that run the real engine load their config through `shuttleengine.LoadConfig` and so pick up the template keys.
- The wrap order (fork-subagent, then TTL, then `CLAUDE_ENV_FILE`) keeps `CLAUDE_ENV_FILE` leading both lines, and `sh.WithEnv` quotes the value as the plan's test rows assume.
- The attack-surface question is answered where it arises: card 3's `**Intent:**` states what the refusal blocks and what bounds it, and card 1 states that `LoadConfig` adds no validation.
- `## verify:` runs in the template's order and covers every targeted package, with the `integration` tier run for the packages that carry integration-tagged tests and the `llm` and `tmux` tiers compiled.
- `ImpactSummary` carries a conclusion on each `Edit` card, and no card is `Custom`.
