# Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows

`lookup-agents-through-shuttle` · Note · feature, shuttle, codex, loom

## Idea (operator, 2026-11-03)

The Plan, Discussion-Write and darn writers hand lookups to fresh subagents, and `loom.yaml`'s `subagent_model` names their model so it is never hardcoded.
Today the value is a bare model name the Claude Code Agent tool takes, so a lookup agent is always a Claude subagent inside the writer's own session.
The value should also be able to name another engine, such as Codex (note `codex-engine`): such a lookup agent cannot run through the Agent tool, so lym spawns it through shuttle, and it shows in reed as a window of its own, like any other agent.

## Open

- How the writer asks for a shuttle-spawned lookup and gets its answer back: a `lym` verb that spawns the agent and waits for its reply, much as a fork returns only what is relevant (note `planner-fork-lookups`).
- Whether one config value selects both the engine and the model, in model-spec form such as `codex[high]`, while a bare Claude name keeps today's Agent tool path.
- Several lookups at once: each its own strand, and how many reed windows that makes.
