# Claude Code Mode for writer sessions, set per role

`claude-code-mode` · Task · research, claude, shuttle, config · running · Drift-Recheck

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

## First step: a bench on the Plan writer (operator, 2026-11-16)

The Plan and Discussion writers have most to gain: they explore most before they write.
The bench starts with the Plan writer, whose input is fixed (the approved decision record and the base commit) and which needs no operator answer;
the Discussion writer follows only if the Plan writer shows a gain, since replaying it needs the landed run's answers.

It runs as a darn `docs` task, the way `docs/research/loom/plan-writer-bench-2026-11-12/report.md` ran its arms:
a pair seeded at the landed run's base commit with its frozen decision record, the real Plan-Write row run in it, and the pair stopped after that row.
Only the arm's setting differs from today's Plan-Write; nothing lands, and every pair is removed at the end.

- Inputs: two landed tapestry runs with a twill-free Plan-Write if any, of different sizes.
- Arms, one Plan-Write each per input, six sessions in all:
  - A, control: today's Plan-Write, run fresh in the same harness, since the landed session ran with a live parent and other conditions.
  - B, Code Mode: A with `CLAUDE_CODE_EXPERIMENTAL_CODE_MODE=1` on the writer's launch.
  - C, pipelines: A with the edit directive's read rule lifted, so the writer may read and search with one shell pipeline per call.
    C tells whether a gain comes from Code Mode or only from letting the writer batch in the shell.
- Figures per session, from `tools/tokencount` and the transcript: turns, tool calls (Code Mode's `REPL` calls counted with the calls inside them), lookup subagents, cost, wall time and peak context.
- Quality per plan: one Plan-Burler review round on it with today's reviewer, its findings by severity, the card count, and the files the plan names against the files the landed run changed.
- What the harness needs: a way to set the arm's setting on the writer's launch only.
  If lym has none, the bench's Discussion finds the narrowest one (a launch env for one strand) and names it; it changes no production code.
- One sample per arm and input: a direction, not an effect size; the report says so.
