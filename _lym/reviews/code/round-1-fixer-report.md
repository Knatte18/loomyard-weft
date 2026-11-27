---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Fixer report — code round 1

## Changes

- **F1** (`9b985d87d`): the premise holds; the installed bundle's `subagentPromptCacheTtl` settings text says the variable covers everything outside the main conversation.
  Widened the wording in `internal/shuttleengine/claudeengine/command.go` (both subagent TTL docs), `doc.go`, `docs/overview.md` and `_lym/landing/summary.md`.
  The summary edit is not committed, since this round runs no git command against `_lym`.
- **F2** (`9b985d87d`): scoped the `promptCacheTTLEnvKey` doc to the main conversation, keeping the `FORCE_PROMPT_CACHING_5M` clause.
  Scoped the same claim in `doc.go`, `docs/overview.md` and the `template.yaml` comments of `prompt_cache_ttl` and `prompt_cache_ttl_roles`.
- **F3** (`5048218c5`): the `TestOpenLaunch` doc names both TTL exports.
- Self-check follow-ups: the `config.go` field docs (`114d0e18b`) and the subagent TTL variable's doc (`d8110d04d`).

`lym gate test ./internal/shuttleengine/claudeengine` passed (tier0, 0 failed) before the first commits and again before the self-check commits.

## Self-check

- fix-code-correctness: no findings. Its out-of-lens remark on the mid-phrase break is the fix-consistency item below.
- fix-consistency, LOW, `config.go` TTL field docs left unscoped: fixed as proposed (`114d0e18b`).
- fix-consistency, LOW, mid-phrase line break in the `subagentPromptCacheTTLEnvKey` doc: fixed as proposed (`d8110d04d`).
- fix-affected-set, LOW, `config.go` TTL field docs: duplicate of the first fix-consistency item, fixed as proposed.
- fix-proposal, LOW, "every request outside the main conversation" is slightly broad:
  fixed as proposed (`d8110d04d`), by one clause in the constant's doc saying Claude Code counts a few helper requests as the main conversation.
  The other sites keep Claude Code's own meaning of "main conversation".

## Disputed

None.

## Deferred

None.

## Rulings

None.
