---
verdict: APPROVED
---

# Round 1 review

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`), the six card commits.
I read the whole diff as code and ran `lyx gate test` over `internal/shuttleengine/claudeengine`, `internal/shuttleengine`, `internal/planparser`, `contracts/stencils` and `internal/websterengine`; all pass.

No finding.
Each focus question, settled:

- **Card 1.**
  `buildLaunchCmd` and `buildResumeCmd` both wrap `bashMaxTimeoutEnvKey` with `bashMaxTimeoutMS`, derived from the one `bashMaxTimeout` constant, between the `CLAUDE_ENV_FILE` and prompt-cache-TTL assignments.
  `steerRawGoDeny` and `noticeRawGoDeny` are `var`s whose text matches the card's wording exactly.
  The timeout value, the quick-check rule and the `cd` rule appear in Go only in that steer, plus the `claudeengine` doc and test text that describe it.
- **Provider seam.**
  `BASH_MAX_TIMEOUT_MS` and the Bash `timeout` appear in Go only under `internal/shuttleengine/claudeengine`.
  Outside it (`internal/shuttleengine/doc.go`, `internal/websterengine/doc.go`, `internal/planparser`) the text says "the gate rule in the system prompt" and states no value.
- **Card 2.**
  `forkActivity` reads `<session>/subagents/*.jsonl`.
  A pending `tool_use` reads `now` only while `now` is no later than the fork's last write plus `bashMaxTimeout`, so a fork stopped mid-call reads active for at most the ceiling.
  A fork older than the ceiling is never scanned.
  The API-error part still reads the main transcript only.
  The new rows in `TestTurnEndActivity` cover each branch: a later fork write, a pending call within the ceiling, a sibling result, a pending call beyond the ceiling, and an older fork with all results.
- **Cards 3 to 6.**
  Each stencil, spec row, refusal detail and `CLAUDE.md` rule points at "the gate rule in your system prompt" and restates no timeout value, quick-check rule or `cd` rule.
  The implementer and verify-fix bodies no longer spell `run_in_background` or ``from `{{.worktree_root}}` ``, and the claims `wantNone` rows pin that.
  `grep` for "background Bash call" finds no leftover text about `lyx gate test` outside `_lyx/`.
  The remaining "backgrounded" wording in `webster-template-master.md` and `internal/websterengine/doc.go` is Merriam's own backgrounded `recover-batch` and fork, which the plan keeps.
- **Card 4.**
  The Master template reads an interim fork or fixer notice as no completion, adds the no-second-fixer line, and the idle-turn-end sentence now matches the hold behavior and the card 2 reading.
- **Test economy.**
  No new top-level test was added: every card extends `TestTurnEndActivity`, `TestBuildLaunchCmd`, `TestBuildResumeCmd`, `TestBuildDenyNotice_SentenceContent`, `TestValidate_VerifyModuleWide` or a claims entry.
  No package row in `cmd/lyx/testdata/test-budget.yaml` is passed, so no budget edit is owed.
- **Per-card mechanical checks.**
  The card types are `Create` and `Edit` only, so the `Rename` and `Delete` mechanical checks do not apply.
  The edits compile, vet, and the touched packages' tests pass.
