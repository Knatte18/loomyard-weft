# Keep a fork's gate call in the foreground past 180 seconds

A fork running `lym gate test` in the foreground, with a one-hour Bash timeout, was moved to the background after 180 seconds.
The call then answered exit 75 to a call nobody was waiting on, and had to be run again.

Claude Code backgrounds a background subagent's foreground Bash call after 180 seconds unless that subagent's prompt cache is one hour.
Forks ran on the 5-minute default, and a capped gate call waits up to the role's keep-warm cap (250s by template).

The claude engine's launch, resume and open lines now set `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` to `1h`.

Closes #275

## Design decisions

- **Raise the subagent TTL rather than lower the keep-warm cap.**
  A fork's gate call stays in the foreground for the whole cap, and a cap above five minutes no longer leaves forks' caches cold.
- **Cost.**
  The variable covers every request outside the main conversation: subagents, forks, and background and helper requests such as compaction.
  All of them now pay one-hour cache-write prices.
- **Not configurable.**
  The value is a constant, applied to every role.
  `claude.prompt_cache_ttl` and `claude.prompt_cache_ttl_roles` govern the main conversation only, and the docs and template comments now say so.

## To check by hand

- No live session was run to confirm that a fork's `lym gate test` call now stays in the foreground past 180 seconds.
  Check it from a freshly launched strand.
  A strand launched before this change resumes on its stored line, which lacks the variable.
- The scope of the one-hour subagent TTL was understated as subagents and forks only.
  Check that the wording on `subagentPromptCacheTTLEnvKey` and `subagentPromptCacheTTL` in `internal/shuttleengine/claudeengine/command.go`, in that package's `doc.go` and in `docs/overview.md` states the wider scope correctly.

## Merge danger

**Door:** two-way
**Blast radius:** shuttle
