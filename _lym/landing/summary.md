# Measure what the orch and spawned roles load at start, by source

Adds a research study, `docs/research/tokens/startup-context-2026-11-05/`, that breaks down the start context of the hub orch and two spawned roles by source, and names how to leave out what each does not need.
It changes no code or config: the report proposes, and the operator picks what becomes a task.

## What was measured

Claude Code records every API request in the session transcript: the system prompt and tool schemas in an `api-request-shape` record, and the message blobs each request keeps and adds.
`measure.sh` rebuilds one recorded request from those records, and `classify.jq` sorts each line into the source that put it there: Claude Code's system prompt and tools, claude.ai connectors, each plugin's skill listing, preloaded skill bodies, instruction files, auto-memory, hooks and lym's own injections.
Token figures split the request's measured input tokens in proportion to characters.

Three sessions, each once, from their recorded transcripts:

- A spawned role, the darn writer and a Burler reviewer alike, reaches its role-prompt turn at 34,252 tokens.
  About 11,000 of them are sources no writer or reviewer uses: the claude.ai connectors (~2,100), the `anthropic-skills` and bundled skill listings (~6,300), auto-memory (~1,400), and unused plugin skills and tools.
- The orch's first request after a compaction is 45,865 tokens, of which about 7,900 can be left out by settings.
  Reading its role file, `.lym/orch/role.md`, adds 19,390 tokens each time, more than all of Claude Code's own start load.

## Proposals in the report

Each lever goes in the settings file lym already passes with `--settings`, or on the launch line, so lym can set it per role: `disableClaudeAiConnectors`, `disableBundledSkills`, `skillOverrides`, `autoMemoryEnabled: false` for spawned roles, and `--disallowedTools ListAgents`.
Together they would take a spawned role's start from 34,252 to about 23,800 tokens.
The settings are named in the Claude Code 2.1.318 binary; none was tried in a launch.
The report also proposes shortening the orch role file or moving its rarely used parts behind an on-demand read.

## Design decisions

- The study reads existing transcripts instead of launching fresh sessions, so no orch or run was disturbed; each counts as one fresh start of its session or cycle.
- The orch is measured at its latest compaction cycle, since it has not run `/clear` since 2026-10-25; its `/clear` start is listed as not covered.
- The scripts are shell and jq, so they run in a lym session.
