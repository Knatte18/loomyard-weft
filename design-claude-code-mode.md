# Claude Code Mode for writer sessions, set per role

`claude-code-mode` · Note · enhancement, claude, shuttle, config

## Seen (operator, 2026-11-16)

Claude Code's Code Mode (https://code.claude.com/docs/en/code-mode, experimental, off by default) lets the model write a JavaScript program that makes several tool calls in one step:
commands run side by side, one result feeds the next call, and only the part of the output it needs comes back.
lym's transcript audits found writers spend many turns on serial single reads and searches,
and the batching sentences in the stencils changed nothing (`docs/research/turns/batching-effect-audit-2026-11-01/report.md`);
Code Mode batches in the tool instead of the prompt.

What the page says that bears on lym:

- It is turned on by `CLAUDE_CODE_EXPERIMENTAL_CODE_MODE=1` in the process environment or user settings; project settings are ignored, and a `0` in either place wins.
- It runs only in an interactive terminal session, which lym's tmux panes are; not with `-p`, the Agent SDK, the VS Code panel or Desktop.
- It needs Claude Code v2.1.323 or later (Neve has 2.1.325), a subscription or Console account, and the 5.5 models or Fable 5.2.
- It rules out Remote Control, agent teams, OpenTelemetry export, `DISABLE_TELEMETRY` and an API key helper.
- A Code Mode session resumes only with Code Mode on.
- Code that does not run in the background stops after ten minutes.
- Every tool call the code makes still goes through `deny` and `ask` rules and `PreToolUse` hooks; the code itself runs as a tool named `REPL`.
- The first session asks "Yes, turn on Code Mode" and restarts.

## Idea

Code Mode as a per-role setting beside the model, off by default:

- lym sets the variable on the launch command of a session whose role turns it on, as it sets `CODEX_HOME` for Codex; never in user settings.
- Candidates: the deep exploration roles first (operator, 2026-11-16), such as the Discussion and Plan writers, their lookup subagents, and darn `docs` writers of research tasks; then the roles that read and edit much, such as the darn writer and Webster's batch implementers.
  Whether a subagent spawned through the Agent tool runs in Code Mode with its session is not on the page; the bench checks it.
  Not the orch: it reads much in its talks with the operator, but Code Mode rules out the Remote Control it keeps (`orch-auto-remote-control`).
  Nor reviewers and other light roles.
- A resume relaunches with the same setting as the session's launch, or Claude Code refuses to resume.
- Shuttle handles the first-use confirmation dialog, or a setup step answers it once.
- `tools/tokencount`, `tools/forkaudit` and the skill-load check learn the `REPL` tool's transcript rows.
- `lym gate test` stays its own call, never inside code, because of the ten-minute stop.

## First step

A bench, run as a darn `docs` task through the real producer with only the setting swapped (the rule in `codex-same-as-claude`):
two landed runs, each replayed once with Code Mode: the Discussion writer of one landed motif and the writer of one landed darn `docs` research task, each compared with the landed session on turns, cost, wall time and its output.
It waits until the real-producer route from `codex-same-as-claude` exists.
