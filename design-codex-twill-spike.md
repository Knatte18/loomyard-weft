# Codex spike 2: a twill-like fan of forks with inherited context

`codex-twill-spike` · Note · audit, shuttle, codex

## Seen (codex spike, 2026-10-24)

Twill and the other fan-outs rest on Claude Code forks that inherit the parent's conversation.
The spike (`docs/research/codex-spike-2026-10-24/report.md`, section `/fork and subagents`) found that a Codex subagent spawned with `fork_turns: "all"` inherits it, while the model's own pick was `"none"`; each subagent writes its own rollout naming its parent; no hook fired for the subagent; the parent waits by polling.
Only one subagent was tried.

## Idea

A small docs spike: a twill-like fan in Codex, 3-4 subagents spawned with `fork_turns: "all"` in parallel, each writing its own file, the parent waiting and merging them.
It answers whether twill can run on Codex: parallelism, the wait, whether a prompt reliably gets `"all"`, cost of an inherited context, and how lyx would audit the forks without hooks (their rollouts).
Same setup as the first spike: a private tmux server, the cheapest model, tiny prompts, report under `docs/research/`.
