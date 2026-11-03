# Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows

`lookup-agents-through-shuttle` · Note · feature, shuttle, codex, loom

## Idea (operator, 2026-11-03)

The Plan, Discussion-Write and darn writers hand lookups to fresh subagents, and `loom.yaml`'s `subagent_model` names their model so it is never hardcoded.
Today the value is a bare model name the Claude Code Agent tool takes, so a lookup agent is always a Claude subagent inside the writer's own session.
The value should also be able to name another engine, such as Codex (project `codex-engine`): such a lookup agent cannot run through the Agent tool, so lym spawns it through shuttle, and it shows in reed as a window of its own, like any other agent.

## Built since (tidy, 2026-11-03)

Part of project `codex-engine` has landed on its beam branch: `codexengine` is a shuttle engine, registered as `codex`, and the model aliases `sol`, `astra` and `luna` resolve to it at `effort: medium` in the built-in registry and the `models.yaml` template.
Codex sessions load skills and get the same shell guard rows a Claude session gets.
A Codex session can run as the assistant reviewer seat of a Burler review round: the `*_assist_reviewer_model` keys of `loom.yaml` and `tapestry.yaml` take a model choice that names a Codex model, and the seat is spawned through shuttle.
The chair waits for the assistant with `lym shuttle await <end marker>` and reads its report as advice; a Codex assistant fans its own lens subagents with `spawn_agent` and `wait`.
That is a shuttle-spawned agent on another engine whose result returns through a file and an end marker, a pattern this note's lookup agent can reuse.
No writer lookup runs on Codex yet, and nothing spawns a lookup agent through shuttle.

## Open

- How the writer asks for a shuttle-spawned lookup and gets its answer back: a `lym` verb that spawns the agent and waits for its reply, much as a fork returns only what is relevant (note `planner-fork-lookups`).
  The assistant seat's report file and end marker are one existing answer path.
- Whether one config value selects both the engine and the model, in model-spec form such as `codex[high]`, while a bare Claude name keeps today's Agent tool path; the aliases above already name a Codex model, so the value may take an alias.
- Several lookups at once: each its own strand, and how many reed windows that makes.
