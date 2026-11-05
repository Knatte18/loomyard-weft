# Find what sessions load at start that they do not need, to cut start tokens

`startup-token-trim` · Task · research, claude, orch, shuttle, stencil · draft

## Wanted (operator, 2026-11-05)

Find what a lym session loads at start that it does not need, and cut it, so every session starts with fewer tokens.
The orch matters most, since it cycles its context often and pays the start load again at each cycle, but spawned agents (Discussion, Webster, reviewers, fixers) start far more often and count too.

## What to measure

For the hub orch and for one or two spawned roles, the context tokens at the first turn, broken down by source:

- Claude Code's own system prompt and tool definitions, the deferred tool list included.
- MCP servers and claude.ai connectors: the orch's start lists Gmail, Google Drive, Google Calendar and Claude Docs tools, which no lym role uses.
- Skills listed at start: the orch sees plugin skills from scribe (Python, C# and Go build, comment and testing skills), prowler, and the built-in anthropic-skills and artifact skills, most of which no lym role uses.
- Plugins and their hooks, such as the SessionStart hook that asks for `scribe:conversation` and `scribe:prose`.
- The user's global `~/.claude/CLAUDE.md`, the hub's `CLAUDE.local.md` and `CLAUDE.md`, and auto-memory with its `MEMORY.md` index.
- What lym itself injects: `role.md`, the orch note, heddle's rendered instructions, stencils, and the system-prompt append lym passes to each session.

## Report

- A table per measured session: source, tokens, whether the role needs it.
- For each source a role does not need, the way to leave it out (a Claude Code setting, a launch flag such as a strict MCP config or disabled connectors, a plugin scope, a narrower skill set, a shorter lym injection) and whether lym can set it per role at launch.
- No change to code or config: the report proposes; the operator picks what becomes a task.

## Cap

Measure the hub orch and at most two spawned roles, each once, from one fresh start; no repeats.
