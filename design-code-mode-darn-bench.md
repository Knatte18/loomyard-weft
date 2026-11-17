# Bench Code Mode on a darn writer doing a repeated edit

`code-mode-darn-bench` · Task · research, claude, shuttle

## Seen (orch, 2026-11-16)

The bench `claude-code-mode` found no gain from Claude Code's Code Mode on the Plan writer,
and the web study `code-mode-web-research` (report `docs/research/claude/code-mode-web-research-2026-11-16/report.md`) found nothing that would change that:
Anthropic and users agree Code Mode pays on mechanical, read-heavy work, such as one change repeated across many files,
and not where each result needs a judgment, as in planning.

## Idea

A bench of Code Mode on a darn writer doing mechanical work:
replay one landed darn task whose diff is a repeated edit across packages, control against Code Mode,
and compare turns, cache-read tokens, cost, wall time and scripts that ended in an error.
Before it runs, one Code Mode session checks that lym's shell guard still refuses a guarded command run inside a script.

## Cap

- One input: one landed darn task.
- Two writer runs: one control and one Code Mode, each once, on the darn writer's default model and effort.
- One guard-check session before them.
