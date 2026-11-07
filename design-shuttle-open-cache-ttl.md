# lym shuttle open sessions write a 1h prompt cache instead of their role's 5m

`shuttle-open-cache-ttl` · Note · bug, shuttle, claude

## Seen

Reported by the `comment-cleanup` writer (2026-11-07), not yet checked by the hub orch.
Sessions opened with `lym shuttle open` write their prompt cache with a 1h TTL: their transcripts show `ephemeral_1h_input_tokens`.
`claude.yaml` resolves `prompt_cache_ttl` to 5m for their role.
Sessions started by `lym shuttle run`, and their subagents, write 5m.
`claudeengine.Prepare` resolves the TTL from `Spec.Role` and puts `CLAUDE_CODE_PROMPT_CACHE_TTL` on the launch line; the open path seems not to set it.

## Wanted

A session from `lym shuttle open` writes its cache at the TTL `claude.yaml` resolves for its role, as a `lym shuttle run` session does.
The operator wants 5m for these sessions.
