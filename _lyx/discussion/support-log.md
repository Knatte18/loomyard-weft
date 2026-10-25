## Interview

A darn run holds no interview; the board entry and the operator decisions it records are the whole input.

## Rejected alternatives

- Drive the real creel daemon in the test: rejected, because the board entry tests the hook path before creel changes, and a daemon change would be production work.
- Use `claude -p` for the Claude receiver: rejected by the repository rule that agents run interactive.
- Install the hook in the operator's `~/.claude/settings.json` or `~/.codex/config.toml`: rejected, because the entry forbids touching the operator's or the plugins' settings.
- Measure token cost of the hook: dropped, because the entry measures time for the no-mail case and the hook adds no tokens then.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the report live? Resolved from the entry: `docs/research/creel-hook-delivery-<date>/report.md`, dated 2026-10-25.
- How is Codex given its hooks without editing the operator's config? Self-pick: `-c hooks.…` flags with `--dangerously-bypass-hook-trust`, as the spike found.
- How is Claude given its hook without editing user settings? Self-pick: a settings file under `.scratch/` passed by `--settings`.
- Which models? Self-pick: Haiku for Claude and `gpt-6-luna` low for Codex, to keep quota use small.
