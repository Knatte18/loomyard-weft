# Research: Codex CLI hands-on spike report

Adds `docs/research/codex-spike-2026-10-24/report.md`, the result of driving Codex CLI 0.164.0 by hand in a private tmux server.
It is step 1 of the note `codex-engine` and of `docs/research/codex-feasibility.md` section 3; nothing is built and no code changes.

The report covers each item of the task: startup and the trust prompt, idle and busy pane readings, bracketed paste and the swallowed Enter after typed text, Esc interrupt, payloads of the `SessionStart`, `UserPromptSubmit`, `Stop`, `PermissionRequest` and `PreToolUse` hooks, the rollout file with its `token_count` event, model and effort selection, approval and sandbox behavior, `AGENTS.md` and `CLAUDE.md` loading, skills in `.agents/skills`, and `/fork` and subagent inheritance.
It ends with a proposed launch line, a pane classifier table and a send sequence for a future `codexengine`.

Findings a `codexengine` design has to respect:

- `-a never -s workspace-write` stops every approval prompt, but the operator's `~/.codex/rules/` allow rules (`bash`, `sh`, `git` and others) run matching commands outside the sandbox, and the interactive `codex` has no `--ignore-rules`.
- Trust cannot be pre-set with `-c`; Codex writes it to `~/.codex/config.toml` when the prompt is accepted, so the engine's startup classifier presses Enter on it.
- Typed text and Enter sent in one burst loses the Enter; a bracketed paste followed by Enter submits.
- An interrupted turn fires no `Stop` hook, so completion needs the `Stop` hook or `task_complete`, and interrupt needs `turn_aborted`.
- `PreToolUse` and `PermissionRequest` hooks can deny, which carries the shell-guard table; a hook that fails to run fails open.
- `CLAUDE.md` loads through the fallback setting only where no `AGENTS.md` exists.
- `/fork` inherits the conversation; a subagent does so only when spawned with `fork_turns` set to `all`.

## Design decisions

- The spike ran on `gpt-6-luna` at low effort with one-line prompts to spend little quota.
- Codex wrote one `projects."<dir>"` trust entry to `~/.codex/config.toml` when the trust prompt was accepted, since `-c` could not pre-set it; the report names the entry so the operator can delete it.
- The throwaway repository, hook scripts and probe captures stayed under the git-ignored `.scratch/`; only trimmed samples are in the report, with account ids left out.
- Items not exercised (`Tab` queueing, a `PermissionRequest` `allow` decision, `~/.agents/skills`, `codex exec`, billing terms, hook "Trust all") are listed in the report's `Not tested` section.
