# Round 1 fixer-report

## Changes

- **F1** (fixed as proposed): deleted the `claude_deny_agent_tool` row from `KeyMoves` in `internal/configreg/keymove.go`.
  Added a row to `TestReconcile_MovesKeysAcrossModules` showing a shuttle layer holding the key reconciles without writing `deny_agent_tool` into `claude.yaml`.
  The other cases already used `deny_ask_user_question`.
- **F2** (fixed as proposed): `headlessClaudeCommandPattern` now takes `(-[a-z]*p[a-z]*|--print)`.
  Added deny rows `claude -pc hi` and `claude -cp hi` and an allow row `claude -c` to the integration table, and regenerated the 16 pattern copies in `testdata/settings-golden.json`.
  Follow-up from the self-check: the row comment in `rows.go` and the paragraph in `claudeengine/doc.go` now describe the wider match and the wider false denial.
- **F3** (fixed as proposed): deleted the `hasMatcher("Agent")` check from `TestBuildSettings_DenyToggleMatrix`, and the `hasMatcher` helper it left unused.

Gate repair: the F2 row comment in `rows.go` carried a fixed-column-wrapped line, so no test ran.
I rewrote it as one sentence per line, with the false-denial cases split into two sentences.

Commits: one per finding, plus one for the F2 documentation follow-up and one for the gate repair.
The gate was green on the touched packages after each change.

## Self-check

- fix-consistency, LOW: `claudeengine/doc.go` still said the row denies `-p` or `--print` — fixed as proposed.
- fix-consistency, LOW: the widened false-denial class was undocumented — fixed as proposed, in `doc.go` and the `rows.go` row comment.
- fix-proposal, LOW: the pattern also refuses a single-dash `p` word of a command launched through a claude subcommand — merged with the finding above; documented as an accepted false denial, not tightened, since the row is a guardrail and the old pattern had the same reach for the word `-p`.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

The `Forms` of the headless-claude row (`{"claude","-p"}`, `{"claude","--print"}`) were left as they are; they name the row's representative spellings, not its full match.

## Disputed

None.

## Deferred

None.
