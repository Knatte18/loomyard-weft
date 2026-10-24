# Codex CLI hands-on spike: tmux driving, hooks, approvals, report

`codex-spike` · Task · audit, shuttle, codex

## Task (operator, 2026-10-24)

Hands-on spike of Codex CLI as a future shuttle engine, step 1 of note `codex-engine`; a report only, nothing built.
Codex 0.164 is installed (`~/.local/bin/codex`) and logged in with the operator's ChatGPT account.

- Drive Codex only in a private tmux server of your own (`tmux -L codex-spike`), never in the `loomyard` session, and in a throwaway directory under `<task worktree>/.scratch/`.
- Launch it with `-a never -s workspace-write` (or `-c` overrides) so it asks no approvals; never edit `~/.codex/config.toml` or `~/.codex/rules/`.
- Test what `docs/research/codex-feasibility.md` section 3 step 1 and note `codex-engine`'s "What to test" list: startup, trust prompt, idle and busy readings in the pane, bracketed paste, Esc interrupt; hook payloads (SessionStart, Stop, UserPromptSubmit, PermissionRequest) captured through a per-run `-c` hook config; a session file under `~/.codex/sessions/` with its `token_count` events; model and effort selection; approval and sandbox modes, including whether `-a never -s workspace-write` stops every prompt and what it blocks (network, writes outside the workspace); `AGENTS.md` loading; skills in `.agents/skills`; whether a subagent or `/fork` inherits the conversation.
- Keep prompts to Codex tiny: the spike spends little of the operator's Codex quota.
- Write `docs/research/codex-spike-2026-10-24/report.md`: what was seen for each item, pane captures and payload samples trimmed to what a `codexengine` needs, and what the engine's launch line and classifiers would be.
- The run changes no code outside the new folder.
