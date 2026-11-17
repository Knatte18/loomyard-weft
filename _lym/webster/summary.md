# Codex session gets Claude's standing items, user setup, instruction files and subagent terms

All three batches recorded `status: done`; none needed recovery, and no digest listed renames, departures or warnings.

## Batch 01-claude-imports-leaf (head `f1c5622d`)

Covers cards 1 to 4: the `internal/claudeimports` leaf, the shared `agents-prompt` target, the hub reconcile render of heddle's file targets, and `AGENTS.override.md` carrying AGENTS.md and the import-expanded CLAUDE.md.
Scope-guard deviations: `internal/claudeimports/claudeimports.go`, `claudeimports_test.go`, `testmain_test.go`, `internal/fabricengine/worktreeseam_integration_test.go`, `internal/heddleengine/importrecord.go` and `internal/hubreconcile/ensure_integration_test.go`.

## Batch 05-codex-developer-instructions (head `48435b58`)

Covers cards 5 to 7: Codex standing items in `developer_instructions` with the 128 KiB refusal, the Codex subagent terms, and the `shuttleengine` user-setup seam with claudeengine's user setup source.
Scope-guard deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/shuttleengine/claudeengine/usersetup.go` and `usersetup_test.go`, `internal/shuttleengine/codexengine/developerinstructions.go` and `prepare_test.go`, `internal/shuttleengine/usersetup.go`.

## Batch 08-plugin-session-start-contexts (head `9c007c70`)

Covers cards 8 to 10: plugin `SessionStart` contexts, the Codex engine taking the user setup, hook runner and sandbox probe as seams, and `project_doc_max_bytes` over the files Codex reads.
Scope-guard deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/heddlecli/verbs.go`, `internal/shuttleengine/claudeengine/pluginhook.go`, `pluginhook_integration_test.go` and `usersetup_test.go`, `internal/shuttleengine/codexengine/instructionfiles.go`, `usersetup.go` and `usersetup_test.go`.

## Verify gate

The plan-level gate runs after this turn ends; no findings so far.
