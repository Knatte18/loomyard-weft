```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that share state, keys or invariants, then checked the criteria and constraints against those decisions.

**Pairs checked**
- D1 × D4 (one spec per engine against the prompt-engine refusal): held.
- D1 × D5 (notices and standing passages moving into `developer_instructions`, read back from the launch): held.
- D1 × D6 (subagent effort and thread cap against D1's compared channels): broken. See F1.
- D1 × D9 (`project_doc_max_bytes` against compared channels): broken. See F1.
- D1 × D8 (plugin contexts delivered or omitted against the faked hook runner): held.
- D1 × D10 (a manifest read back from `Prepare`'s output against "writes nothing"): broken. See F2.
- D1 × D12 (the nudge and the question deny as compared channels): held.
- D2 × D3 (scan scope against the Claude copy under `engine/claude/`): held.
- D2 × D4 (orch places skipped against the orch pinned to Claude): held.
- D2 × D6 (Codex subagent passage text against the Go-literal scan): held.
- D2 × D7 (shipped entries against the scan): held. The entries live in `internal/heddleengine/shipped.json`, outside the scan; D7's `Review:` criterion covers them.
- D3 × D2, through the edit-rule guard: held. `TestStencils_NoEditRuleRestatement` already exempts by base name `edit-directive`, which the engine copies share (`contracts/stencils/parentdirective_test.go:163`, `stencilstore.EngineBlockName`).
- D3 × D4 (engine passed to `editdirective` against the prompt engine set from the same model choice): held. Today assist seats pick their stencil engine from `geom.DefaultEngine` (`internal/seatengine/prompt.go:73`), and D4 turns a mismatch into a refusal.
- D5 × D8 (the order of developer-instruction items): broken. See F3.
- D5 × D9 (instruction-file pointers in the order): held.
- D5 × D12 (the creel trust sentence against the nudge delivered as a block `reason`): broken. See F4.
- D7 × D5 (`agents-prompt` into `codex-prompt`, which D5 routes into `developer_instructions`): held.
- D7 × D9 (the override file is both heddle's rendered `agents-file` target and D9's carrier for the repo's `AGENTS.md` and `CLAUDE.md`): held as stated. Both render through heddle.
- D8 × D9 (one shared `@`-import helper; the global file moves to developer instructions, outside the cap): held.
- D10 × D8 (no hooks run; contexts listed by plugin and hook): held.
- D11 × D4 and D11 × D10 (the bench runs the real producer with `run_settings`; the report uses the verb): held.
- D12 × D6 (the `PreToolUse` deny covers children, per the twill spike): held.
- D12 × creel's existing `PostToolUse` mail hook (the nudge must join the one `hooks.PostToolUse` setting): broken. See F4.

**Findings**

- **F1.** MEDIUM. Class: consistency. Location: Acceptance criteria, first bullet, against D1, D6 and D9.
  - The criterion says dropping any Codex delivery of D6 or D9 fails the parity test.
  - D1 lists engine settings and does not compare them. D6's `default_subagent_reasoning_effort` and thread cap, and D9's `project_doc_max_bytes`, are Codex `-c` config settings.
  - Claude's counterpart is also a setting: the settings environment from `subagentEffortEnv`.
  - So dropping any of them changes no compared item, and the criterion cannot hold as D1 is written.
  - A truncating cap also leaves the instruction-source items in the launch manifest intact: the manifest is read from the launch, not from what Codex loads.
  - Evidence: `decision-record.md` lines 33, 95–99, 135 and 193; `internal/shuttleengine/claudeengine/subagents.go:34–40`.
- **F2.** MEDIUM. Class: design. Location: D10's bound against D1.
  - D1's manifest is "read back from the launch its `Prepare` produced", and `Prepare` "writes the manifest into the run directory".
  - The Notes say both engines' `Prepare` writes launch files.
  - D10 shares the test's code yet promises it "writes nothing", and names no scratch location or cleanup that would make that true.
  - Evidence: `decision-record.md` lines 27, 34, 146, 148 and 218.
- **F3.** LOW. Class: consistency. Location: D5's order against D8.
  - D8 counts the skill listing and the plugin session-start contexts as part of "the user's Claude setup" (its bullets 3–4).
  - D5 orders "the user setup of D8, the instruction-file pointers of D9, the skill listing and the plugin session-start contexts".
  - So the listing and the contexts sit in two places in the stated order.
  - Evidence: `decision-record.md` lines 85 and 119–124.
- **F4.** MEDIUM. Class: design. Location: D12 against D5 and creel's Codex hook.
  - D12 hands the nudge over as a `PostToolUse` block `reason`.
  - On Codex, a block after a tool call makes the model read that call as failed, so the session may run it again. Codex reads a block reason as a failed call unless its instructions say `[creel]` lines are mail.
  - D5's creel trust sentence covers only `[creel]` lines, not the nudge line. A nudge meant to cut calls can therefore prompt a repeated call.
  - The nudge must also share the one `hooks.PostToolUse` setting with creel's mail hook, because a second `-c` of the same key replaces the first. No recorded fact says how Codex merges two block outputs from one tool boundary.
  - The one alternative channel on record: Codex took `additionalContext` with `systemMessage` in the creel spike. Claude's nudge already uses `additionalContext`.
  - Evidence: `internal/shuttleengine/codexengine/doc.go:64,110`; `codexengine/command.go:234,247–251`; `docs/research/creel/creel-midturn-trust-2026-10-28/report.md:198–206`; `claudeengine/batchingnudge.go:297`.

**Tool calls**
- `claudeengine/subagents.go` head, `lym-batching` locations, and `seatengine` `stencilEngine`: to check whether Claude's subagent effort is a setting (F1), where the shipped entries live (D2 × D7), and which engine an assist seat renders for (D3 × D4).
- grep for the shipped entries' definitions: to confirm they sit in `shipped.json` and outside D2's Go-literal scan.
