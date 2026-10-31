# A task that turns out to be a project is refiled as a project, keeping its Discussion findings

`beam-promote-keeps-discussion` · Note · enhancement, workspaces, batten, loom

## Seen (2026-10-25, operator 2026-10-30)

A beam is made from a project entry with `lym beam create`, and its life is one batten run over that entry with the `pawl` recipe.
`lym beam promote` is removed: switching a running task's worktree into a beam halfway proved messy.
A task can still turn out too large while its Discussion runs, and then it has to become a project instead.

## Idea (operator, 2026-10-30)

The agent that finds this out, in Discussion or in a research or darn run, files a note on the board with everything it found, tells its parent, the orch, and ends its turn.
It never files a project or a task, and starts nothing: only the operator decides what becomes a task or a project.
The orch puts the finding to the operator; when the operator decides on a project, the old run ends and the project is created and spawned as a new beam from the parent, never from the old task's worktree.
Nothing is switched in place.

The note must keep what the agent learned: its body holds the Discussion writer's findings and decisions, so they become the beam's starting design and are not left behind in the old run's records history.
Note `loom-beam-darns` is a related shape, where a large tapestry task turns into a beam and runs its pieces as darn runs inside it.

## Open design points

- How the agent tells its parent: a message to the orch with the note's slug, and whether the run stops by itself afterwards or waits for the orch to end it.
- How the run ends once the operator has decided: the orch ends it with `lym batten cancel <slug> --remove`.
- How the note becomes the project entry and its beam: the orch turns it into a project entry after the operator's go-ahead and runs `lym beam create`, with the old task's entry removed or marked superseded in the same step.
- Whether the old run's code commits, if any, are kept or dropped.
- With dobby as the only driver of a beam's runs (only beams have a dobby, and the prime is the root beam), a worktree that is to have worktrees under it must be a beam from the start, so this route is the only way a task's scope grows into a tree of runs.
