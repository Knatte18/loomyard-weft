# Terminal titles read hub: project: slug

`terminal-titles-hub-project` · Note · enhancement, ide, reed

## Seen (orch, 2026-11-09)

The operator runs several hubs at once on one machine and cannot tell at a glance which terminal belongs to which hub and project.
Today the VS Code `window.title` of a worktree is the upper-cased hub shortname, a colon and the slug (`docs/overview.md`, the `lym ide` paragraph); a project is not in it.
The tmux windows reed opens carry names of their own (`shell`, `overview`, `beam: prime`, `<handle>: beam: <slug>`), with no hub shortname.

## Idea

Every terminal and IDE window title reads `<hub shortname>: <project>: <slug>`, with the project left out when the entry has none.

Exceptions proposed:

- The prime has no project and no slug: `<hub shortname>: prime`.
- A beam is a project and its own worktree: `<hub shortname>: <beam slug>: beam`.
  A task under a beam reads `<hub shortname>: <beam slug>: <slug>`, so the project slot names the beam.
- Strands that are not a task keep their role as the last part: `<hub shortname>: orch`, `<hub shortname>: runs`, `<hub shortname>: shell`.
- A workspace file's title keeps its `workspace` suffix after the hub shortname.

## Task shape

Darn, profile `light`.
It reads the project from the task's board entry (`project` field), once, when the title is written; a task whose project changes later keeps the title until the next spawn.
It changes only where the title string is formed and the tests that pin it; no title is read back by another part of lym.

## Open

- Which windows count: the VS Code window title, the tmux window names, or both.
  The proposal assumes both, from one shared formatter.
- Whether the hub shortname stays upper-case as in the IDE title today.
