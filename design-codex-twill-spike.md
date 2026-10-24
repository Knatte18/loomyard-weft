# Codex spike 2: a twill-like fan of forks with inherited context

`codex-twill-spike` · Task · audit, shuttle, codex · running · PR-Review

## Seen (codex spike, 2026-10-24)

Twill and the other fan-outs rest on Claude Code forks that inherit the parent's conversation.
The spike (`docs/research/codex-spike-2026-10-24/report.md`, section `/fork and subagents`) found that a Codex subagent spawned with `fork_turns: "all"` inherits it, while the model's own pick was `"none"`; each subagent writes its own rollout naming its parent; no hook fired for the subagent; the parent waits by polling.
Only one subagent was tried.

## Idea

A small docs spike: a twill-like fan in Codex, 3-4 subagents spawned with `fork_turns: "all"` in parallel, each writing its own file, the parent waiting and merging them.
It answers whether twill can run on Codex: parallelism, the wait, whether a prompt reliably gets `"all"`, cost of an inherited context, and how lyx would audit the forks without hooks (their rollouts).

## Task (operator, 2026-10-24)

A report only, nothing built: `docs/research/codex-twill-spike-2026-10-24/report.md`.
- Same setup as the first spike: Codex only in a private tmux server of your own (`tmux -L codex-twill`), never in the `loomyard` session, in a throwaway git repository under `<task worktree>/.scratch/`, launched with `-a never -s workspace-write`; the cheapest model and tiny prompts.
- Never edit `~/.codex/config.toml` or `~/.codex/rules/`: give Codex its own `CODEX_HOME` under that `.scratch/` folder, holding a `config.toml` with a `[projects."<repo root>"]` `trust_level = "trusted"` table and a symlink to `~/.codex/auth.json`; that skips the trust prompt (tested 2026-10-24).
- Measure: whether 3-4 subagents run in parallel; how the parent waits and learns each is done; whether the prompt reliably gets `fork_turns: "all"` (try it several times); the token cost of each inherited context from the rollouts' `token_count` events; what the subagents' rollouts show for auditing them without hooks.
- End with what twill's directive would need to say on Codex, and anything that blocks it.
- The run changes nothing outside the new folder.
