# A task that turns out to be a project is refiled as a project, keeping its Discussion findings

`beam-promote-keeps-discussion` · Note · enhancement, workspaces, batten, loom

## Seen (2026-10-25, operator 2026-10-30)

A beam is made from a project entry with `lym beam create`, and its life is one batten run over that entry with the `pawl` recipe.
`lym beam promote` is removed: switching a running task's worktree into a beam halfway proved messy.
A task can still turn out too large while its Discussion runs, and then it has to become a project instead.

## Idea (operator, 2026-10-30)

The agent that finds this out, in Discussion or in a research or darn run, files the project on the board itself and ends its turn.
The whole run then ends, and the project is spawned as a new project and beam from the parent, never from the old task's worktree.
Nothing is switched in place.

What the agent files must keep what it learned: the project entry's body holds the Discussion writer's findings and decisions as the beam's starting design, so they are not left behind in the old run's records history.
Note `loom-beam-darns` is a related shape, where a large loom task turns into a beam and runs its pieces as darn runs inside it.

## Open design points

- How the agent files the project: a run session may file only notes today, since only the operator decides what becomes a task, so the agent may need a verb or a rule for a project entry that carries its findings.
- How the run ends once the agent has filed: its own verb, or an escalation to the orch, which ends the run with `lym batten cancel <slug> --remove` after the operator agrees.
- Who creates the beam from the project entry, and when: `lym beam create` after the operator's go-ahead, with the old task's entry removed or marked superseded in the same step.
- Whether the old run's code commits, if any, are kept or dropped.
- With dobby as the only driver of a beam's runs (only beams have a dobby, and the prime is the root beam), a worktree that is to have worktrees under it must be a beam from the start, so this route is the only way a task's scope grows into a tree of runs.
