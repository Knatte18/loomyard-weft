# Web research: how to use Claude Code Mode well

`code-mode-web-research` · Task · research, claude, codex · running · Describe

## Goal

The bench `claude-code-mode` (report `docs/research/claude/code-mode-bench-2026-11-16/report.md`) found no gain from Claude Code's experimental Code Mode on the Plan writer:
it batched twice the tool calls, but saved few turns, since each `REPL` call needs a `Poll` call.
This task finds out from the web how people use Code Mode well, and whether any of it would change that result for lym's sessions.

## What to find

- What Code Mode is meant for, from Anthropic's own docs, changelogs and issues.
- How users get gains from it: prompt wording, task kinds, settings, the `REPL` and `Poll` pattern, subagents.
- Where users report it costs more or helps nothing.
- For each tip: whether lym's writers could use it, and how a later bench would test it.

## Sources

- The web, through the session's own search and fetch tools.
- Reddit, which Claude's fetch refuses: fetch Reddit threads through Codex with `lym shuttle run --model 'astra[high]'`,
  having the Codex session download the threads and save their text under this task's `.scratch/`, then read them yourself.
  Cap: at most 3 Codex sessions in all.
- Record each source's URL and date in the report.

## Bounds

Read and fetch only; change no production code and nothing under Bolt, and run no bench.
The report goes under `docs/research/claude/`, beside the bench report.
