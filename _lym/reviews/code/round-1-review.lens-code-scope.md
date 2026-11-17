```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to a goal point or `Added after Discussion` entry in the decision record (the code-scope lens).

**Decisions (the goal points of the board entry under `## Goal`, plus the Added entries)**
- G1: the `request_user_input` deny on an autonomous Codex launch, in the one `hooks.PreToolUse` setting.
- G2: the Codex batching nudge, which reads the rollout's top-level calls and exec cells, applies Claude's rules and keys counts by `transcript_path`. It sits in the one `hooks.PostToolUse` setting.
- G3: the read-only lookup check moves out of claudeengine into a package under `internal/shuttleengine` that both engines import. The goal's Exports line names that package.
- G4: codex.yaml's `batching_nudge` block.
- A1 (arbiter, 2026-11-16): the assist-trial schedule risk has no subject. This entry asks for no code.
- A2 (arbiter, 2026-11-16): the round-3 findings are answered by the record as it stands. This entry asks for no code.

**Each change and what it traces to**
- C1: `claudeengine/bashlookup.go` moves to `bashlookup/bashlookup.go`, `IsReadOnlyLookup` is exported, and `LineRangeFile` is added → G3. `LineRangeFile` serves G2's line-range read rule.
- C2: the claudeengine call sites now call `bashlookup.IsReadOnlyLookup` and the moved file helpers (`claudeengine/batchingnudge.go`) → forced by C1 and C3.
- C3: `shuttleengine/filetail.go` adds `ReadFileTail` and `CountLinesUpTo`, moved from claudeengine. `TestReadTranscriptTail` moves to `filetail_test.go` and `TestCountLinesUpTo` is new → G2. These are the bounded reads the Codex nudge reuses, a sharing the G2 judgment makes necessary.
- C4: the `sharedPackages` exemption in `seam_enforcement_test.go`, its row, and the bullet in `pattern/PATTERN-shuttle-provider-seam.md` → forced by C1. Without the exemption a new package under `internal/shuttleengine/` breaks the scan.
- C5: `bashlookup/testmain_test.go` → forced by C1. A new test package needs `tmuxkit.Main` under `PATTERN-test-isolation`.
- C6: `questionToolPattern`, `questionDenyCommand` and the steer and notice constants. Also `denyNotice(questionDeny, rows)`, `toolHooks.questionDeny`, and the PreToolUse ordering in `hookSettings` (`guards.go`, `command.go`, `codexengine.go` Prepare) → G1.
- C7: `codexengine/batchingnudge.go` (judgment, state file, hook command, rollout reading), `rolloutWait` on `Codex`, the state seeding in Prepare and the PostToolUse entry in `hookSettings` → G2.
- C8: `rollout.go` splits out `parseRolloutEvents` and `latestContext` → forced by C7, which reads the context from the tail.
- C9: `config.go` `BatchingNudge`, its `ConfigSpec` key and `validateBatchingNudge` in `LoadConfig`, plus the `template.yaml` block → G4.
- C10: `codexengine/doc.go` text on the deny and the nudge, and the package list in `docs/overview.md` → forced by C1, C6 and C7.
- C11: tests (`batchingnudge_test.go`, `TestQuestionDenyCommand_DeniesAQuestionTool`, `TestPrepare_QuestionDenyAndBatchingHook`, and the new rows in `TestHookSettings`, `TestGuardEntries…`, `TestLoadConfig_*` and `TestBatchingNudger`, plus the updated expectations in the Prepare tests) → cover C6, C7 and C9.

**What builds each decision**
- G1 ← C6 (with C10 and C11)
- G2 ← C7, C3, C8, and C1's `LineRangeFile` (with C10 and C11)
- G3 ← C1, C2, C4, C5
- G4 ← C9
- A1 ← no change; none is asked for.
- A2 ← no change; it confirms the record as written, which C7 implements.

**Findings**
None. Every change traces to a goal point or is forced by one that does, and every goal point is built. The record's `## Scope` section lists `internal/shuttlecli`, which the diff leaves untouched. That section is the writer's own claim, not the answer key. The summary says the `hook-batching` verb needed no change because Codex now implements `BatchingNudger`, and `shuttlecli/hookbatching.go` shows that holds. So this is not a scope gap.

Tool calls: none. The inherited context already held the decision record, the full diff and the commit log.
