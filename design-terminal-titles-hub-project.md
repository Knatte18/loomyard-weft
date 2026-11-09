# Terminal titles read hub: project: task: slug

`terminal-titles-hub-project` · Task · enhancement, ide, reed · running · Drift-Await

## Seen (orch, 2026-11-09)

The operator runs several hubs at once on one machine and cannot tell at a glance which terminal belongs to which hub and project.
Today the VS Code `window.title` of a worktree is the upper-cased hub shortname, a colon and the slug (`docs/overview.md`, the `lym ide` paragraph); the project and the task handle are not in it.
The tmux windows reed opens for orch and beam sessions carry names with no hub shortname.

## This task

Every console window the operator sees gets the title `<hub shortname>: <P#>: <T#>: <slug>`, where a part that does not exist is left out.
`<hub shortname>` is lower case, as `lym creel hubs` prints it, also in the VS Code title.
`<P#>` is the handle of the entry's project and `<T#>` the task's own handle, both read from the board when the window is opened.
A task under a beam takes the beam's handle as its `<P#>`.

| What | Title |
|---|---|
| Task in project P3 | `ly: P3: T7: gate-shard-slow-package` |
| Task with no project | `ly: T12: adopt-fix-root-t1` |
| Task under a beam that is project P2 | `mo: P2: T4: wellbore-build-cost` |
| Beam P2 itself | `mo: P2: beam` |
| Prime | `ly: prime` |
| Orch | `ly: orch` |
| Orch in a beam | `mo: P2: orch` |

A workspace file's title keeps its `workspace` suffix after the hub shortname.
The `runs` strand and the operator shell are not consoles of their own, so they get no title of their own.
The title is written when the window is opened and is not rewritten when a handle changes later; a running task is frozen, so its handles do not change under it.

## Limits

- One shared function forms the title for the VS Code window and the tmux windows; the places that form a title today call it.
- Only where a title string is formed, and the tests that pin it, change.
- No other part of lym reads a title back; a part that does is found first and named in the landing note.
