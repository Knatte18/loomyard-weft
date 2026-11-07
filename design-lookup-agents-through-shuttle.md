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

## Built since (tidy, 2026-11-04)

Task `codex-trial-1-fixes` (`cd897ef0`) made `lym shuttle run --model` take the model spec `lym shuttle open` takes, `<alias>[<effort>]` or `<engine>:<model-id>[<effort>]`, both through one `resolveModelSpec`.
The registry picks the engine from the alias, so `lym shuttle run --model 'luna[high]'` runs on Codex whatever `shuttle.yaml` names, and the `--effort` flag is gone.
A shuttle-spawned lookup agent on Codex therefore needs no new engine selection in shuttle: a spec string is enough.

## Measured since (tidy, 2026-11-04)

Task `planner-fork-lookups` (`482c05d4`, `docs/research/turns/writer-lookups-measure-2026-11-04/report.md`) measured how the Plan and Discussion writers use lookup subagents, on 7 writer sessions of 3 tapestry runs, with no agent run and no code change.
- The 3 Plan writers each handed one batch of 3 or 4 `sonnet` lookup subagents to the Agent tool; 1 of 3 full Discussion sessions did.
- Exploration is 46% of the writers' cost ($11.03 of $23.76), and 15% of it is paid above 200k context, all in Plan sessions after their one batch.
- Moving every search turn to subagents would save at most about $4.7, one fifth of writer spend; batching the late serial chains in the writer saves more than a subagent does there.
- The report recommends no bench and no stronger stencil now, and a re-run once more tapestry runs have landed.
So the gain a lookup agent on another engine can add is small against what the Claude subagent path leaves.

## Built since (tidy, 2026-11-05)

Task `discussion-explore-agents-first` (`3085f451`) changed the Discussion-Write stencil: the writer now opens Step 2 with one batch of explore subagents on `subagent_model`, sent together through the Agent tool, before it reads code itself.
Both writer roles now delegate early to Claude subagents, so the gain left for a lookup agent on another engine is smaller still; note `writer-lookups-remeasure` re-measures it.

## Built since (tidy, 2026-11-07)

Task `agent-skills-and-effort` (`e9895522`) moved every role's skill list and effort into the hub config `_lym/config/agents.yaml`.
A launch whose model spec names no effort now gets an explicit one from its `efforts` table, so the `effort: medium` the registry gives the Codex aliases applies only where the spec and `agents.yaml` name none.
A shuttle-spawned lookup agent would take its skills by role from the `skills` key.
Every Claude session's system prompt now tells it which `effort` each Agent-tool subagent call takes, by model, and to name its own fitting skills in the brief.

## Open

- How the writer asks for a shuttle-spawned lookup and gets its answer back: a `lym` verb that spawns the agent and waits for its reply, much as a fork returns only what is relevant (task `planner-fork-lookups`, landed 2026-11-04 as the measure above).
  The assistant seat's report file and end marker are one existing answer path.
- Whether one config value selects both the engine and the model, in model-spec form such as `codex[high]`, while a bare Claude name keeps today's Agent tool path; the aliases above already name a Codex model, so the value may take an alias.
- Several lookups at once: each its own strand, and how many reed windows that makes.
