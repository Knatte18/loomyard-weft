# Board renders root as project P0 with its own file, last in the README

The board README named the entries under no project twice: a `## Prime` wrapper around the real projects, and further down a `### No project` section that looked nothing like a project heading (no link, no facts line, no brief).
The root is now a project like the others, with the handle `P0`, which never clashes because board project handles start at `P1`.

## What changes

- Every project, root included, opens with the same heading: `P<n> <title>` linked to its project file, an open-member count, its facts line and its brief.
  The root's heading is `P0 Root: entries under no project`, and its facts line is `Owner <prime owner>` alone, since root has no board entry to give it a slug, age or design link.
- The root renders to its own `project-root.md`, laid out like any `project-<slug>.md`: Tasks by layer, Notes by type, Done.
- `README.md` stays the full overview with every table it showed before.
  It holds a `##` section per open project under no other, by handle, then the root's `##` section last.
  The `## Prime` wrapper and the `No project` heading are gone, and heading levels are consistent without the wrapper.
- A project file with sub-projects keeps its `Own entries` heading; the root, whose projects stand beside it, has no such heading.
- The board's Bolt replayer still claims `project-*.md`, so `project-root.md` is claimed as the root's file.
- The slug `root` is now reserved: `NewTask`, `ApplyPatch` (only when a patch moves an entry onto it) and `Rename` refuse it, because a project of that slug would overwrite `project-root.md`.
  An entry already stored under `root` stays patchable.

Changes are in `internal/boardengine` (`render.go`, `task.go`, `rename.go`, `doc.go`), their tests and goldens, and `docs/overview.md`.

## Design decisions

- The root is a synthetic `Task` (`rootProject`) with an empty slug and no board entry, so it never appears in `board.json` and has no design file.
- Root's projects render beside it rather than under it: its section and file hold its own entries alone, and its open-member count counts its open tasks and notes, not the projects.
- A file's "up" link goes to the parent's project file, or to `README.md` for the root and any project under no other.

## To check by hand

- Reserved slug `root` in `internal/boardengine/task.go` (`validateSlugNotReserved`, used by `NewTask`, `ApplyPatch` and `Rename`): confirm no existing board on a hub already holds an entry under `root`, and that the carve-out for an already-stored `root` entry in `ApplyPatch` is the behaviour you want.
- Render a real board and look at `README.md` and `project-root.md` once; the goldens cover the layout but not a live hub's data.
