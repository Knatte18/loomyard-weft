# Study: the work to support Gemini CLI or Antigravity as an agent engine

`gemini-engine-study` · Task · research, shuttle · running · Darn-Write

## Question (operator, 2026-11-17)

lym now runs Codex beside Claude Code as an agent engine (project `codexengine`).
How much work is it to support Google's Gemini CLI, or Antigravity, as a third engine?

## Study

- Map what an engine must give lym today, from the code: `internal/shuttleengine`'s engine interface and seams, `claudeengine` and `codexengine`,
  the launch manifest and parity test (`3bbc92a0`), hooks (tool boundary, stop, deny guards, edit recording), idle wake, skills and plugin contexts, instruction files and heddle targets, subagents and forks, session resume, cost and usage records, sandbox and permissions, and headless or pane launch under reed.
- For each of Gemini CLI and Antigravity, read their current public documentation and source where open, and judge per item: native support, a workaround, or missing.
  Name the version read and the date; state what could not be verified from documents alone.
- Use the Codex work as the yardstick: list the `codexengine` tasks that landed and what each cost, and estimate the same work for each candidate in changed lines and runs.
- Say which of the two fits lym better, or whether neither does yet, and what a first minimal level of support would hold.

## Bounds

- A desk study: no code change, no installed tool, no account or API call; web reading only.
- The report goes under `docs/research/` as one file; the verdict on whether to build is the operator's.
