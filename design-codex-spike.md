# Hands-on tests of Codex CLI in tmux

`codex-spike` · Note · enhancement, shuttle

## Idea (operator, 2026-10-13)

Loomyard burns Claude tokens fast, since it spawns hundreds of agents a day; Codex CLI could take part of the load.
First step: short hands-on tests of Codex CLI, since the operator has never used it.

## What to test

From the feasibility study (`.scratch/codex-feasibility.md` in the hub prime):

- Drive the interactive TUI in tmux (`--no-alt-screen`): startup, trust prompt, idle and busy readings, bracketed paste, Esc interrupt.
- Capture hook payloads (SessionStart, Stop, UserPromptSubmit, PermissionRequest) and a session file under `~/.codex/sessions/`, with its `token_count` events.
- Model and reasoning-effort selection, approval and sandbox modes, `AGENTS.md` loading, skills in `.agents/skills`.
- Whether a subagent or `/fork` inherits the parent's conversation.
- Billing: check the current terms for interactive use and `codex exec` under a ChatGPT login.

## First candidate roles

Single-session roles that spawn no forks and only write output files: Burler review chair without a cluster fan, Burler fixer, Describe, Bouncer judge and seed, treadle, friction, merge-conflict resolver, darn writer, the alignment agent.
Webster, twill, the orch and the Burler cluster fan depend on Claude Code's Agent forks, cross-session messaging and background-task wake-ups, and wait.

The engine-neutral groundwork is `llm-agnostic-engine`.
