# Codex CLI as a second shuttle engine: hands-on spike, then codexengine

`codex-engine` · Note · enhancement, shuttle, codex

Codex CLI is the candidate second engine for shuttle, to offload Claude token use.
The operator has never used Codex, so short hands-on tests come first, then a minimal `codexengine`.

## Origin (operator, 2026-10-03)

Operator 2026-10-03, while striking the roadmap's bulk-mode clusters (Burler rounds concatenated into one blob so provider-side caching, Gemini's explicit cache, pays off): that does not work for Claude, and Codex is more relevant to support than Gemini.
Idea: if shuttle gets a second engine, make it Codex.

## Idea (operator, 2026-10-13)

Loomyard burns Claude tokens fast, since it spawns hundreds of agents a day; Codex CLI could take part of the load.
First step: short hands-on tests of Codex CLI, since the operator has never used it.

## What to test

Done on 2026-10-24 except billing: results in `docs/research/codex-spike-2026-10-24/report.md`, with a proposed launch line, pane classifier table and send sequence.
Billing, `codex exec`, `Tab` queueing, a `PermissionRequest` `allow` decision and `~/.agents/skills` were not tested (the report's `Not tested` section).
From the feasibility study (`docs/research/codex-feasibility.md`):

- Drive the interactive TUI in tmux (`--no-alt-screen`): startup, trust prompt, idle and busy readings, bracketed paste, Esc interrupt.
- Capture hook payloads (SessionStart, Stop, UserPromptSubmit, PermissionRequest) and a session file under `~/.codex/sessions/`, with its `token_count` events.
- Model and reasoning-effort selection, approval and sandbox modes, `AGENTS.md` loading, skills in `.agents/skills`.
- Whether a subagent or `/fork` inherits the parent's conversation.
- Billing: check the current terms for interactive use and `codex exec` under a ChatGPT login.

## Permission prompts (operator, 2026-10-24)

In the operator's first hands-on use, Codex asked for approval of nearly every command, and "allow for later" saved an exact command line, not a prefix, so the saved list was useless.
Codex 0.164 has the switches for it: `approval_policy = "never"` (`-a never`) with `sandbox_mode = "workspace-write"` (`-s workspace-write`) never asks and keeps writes inside the workspace, and `~/.codex/rules/*.rules` takes hand-written `prefix_rule(pattern=["git"], decision="allow")` entries.
The spike confirmed `-a never -s workspace-write` stops every prompt, but allow rules in `~/.codex/rules/` run matching commands outside the sandbox and interactive `codex` has no `--ignore-rules` (report, `docs/research/codex-spike-2026-10-24/report.md`).
A `codexengine` sets the approval and sandbox mode on its launch line, as claudeengine does with its permission mode, and never relies on saved approvals.
Operator view, not a decision: Codex's permission list should mirror Claude's, which allows broadly and forbids only destructive commands; a hand translation of `~/.claude/settings.json` sits in `~/.codex/rules/claude.rules` on Neve (2026-10-24).

## Folder trust

The trust prompt is answered by a `[projects."<path>"]` table with `trust_level = "trusted"` in `~/.codex/config.toml`, the entry Codex writes itself on accept; only `-c` cannot set it.
So a `codexengine` writes that entry for the task worktree before launch instead of pressing Enter on the prompt; whether one entry for the hub folder covers every worktree below it is untested.

## Design basis (operator view, not a decision, 2026-10-24)

When claudeengine was built, what an engine needs was unknown; now claudeengine shows it, so `codexengine` is designed from claudeengine's list of needs (launch, startup classifier, send, idle and busy, completion, interrupt, usage, guards), each mapped to its Codex counterpart from the spike report.

## Steps

From `docs/research/codex-feasibility.md`, section 3:

1. Done (2026-10-24, see `docs/research/codex-spike-2026-10-24/report.md`). Spike: drive Codex in tmux and capture startup, trust, idle, busy and paste behaviour, the hook payloads and a rollout file. Small.
2. A minimal `codexengine` for the single-session roles: launch line, hook-fed event parsing, startup and trust classifier, send and paste handling. Medium to large, since claudeengine is about 3,900 non-test lines.
3. Skills sync to `~/.agents/skills`, and the deny guardrails as Codex PreToolUse hooks. Medium.
4. Forks and orch parity. Large; deferred.

## First candidate roles

Single-session roles that spawn no forks and only write output files: Burler review chair without a cluster fan, Burler fixer, Describe, Bouncer judge and seed, treadle, friction, merge-conflict resolver, darn writer, the alignment agent.
Webster, twill, the orch and the Burler cluster fan depend on Claude Code's Agent forks, cross-session messaging and background-task wake-ups, and wait.

The engine-neutral groundwork is the task series of the operator's 2026-10-15 decision, which replaced the note `llm-agnostic-engine`: `engine-registry`, `engine-config-seams`, `engine-model-choice`, `engine-usage-readers` (cost reporting only), and creel in three stages, `creel-core`, `creel-notices`, `creel-agents`, since creel is a prerequisite for Codex.
Everything here that needs Codex installed waits on that series: the `codexengine` itself (rendering the shared guard table as Codex hooks, `codex.yaml`, the registry entry, the engine name, the aliases), the skills sync to `~/.agents/skills`, and tuning.

## Model aliases (operator, 2026-10-15)

The aliases were designed for a second engine: `sonnet[high]` is engine `claude`, model `sonnet`, effort `high`, and Codex's models get the same form, such as `sol[medium]` for engine `codex`, model `sol`, effort `medium` (also Luna, Astra).
