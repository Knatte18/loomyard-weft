# Board README: one rendered file per project, research notes first

`board-readme-per-project` · Task · enhancement, board · running · Describe

## Seen (operator, 2026-10-25)

The board renders one README.md holding every project, task and note of the hub.
The notes are grouped by type label in file order, so after the `audit` label became `research` the research notes moved from first under Notes to last, under the heading `Researchs`.

## Idea (operator, 2026-10-25)

- Research notes come first under Notes, right after the tasks, where the audits stood; the heading reads `Research`.
- Each project gets its own rendered file.
  It opens with the projects directly under it, each a link to its file with its short description, then holds the tasks and notes that belong to the project itself.
- Every project sits under another project, except the root project at the top.
- The split is rendering only: every entry stays in the one board store, and handles stay numbered across the whole hub, so a project's file shows gaps in its numbering and a handle names one entry whichever file shows it.
- A task's After column names its dependencies by handle instead of by slug, one handle per line within the cell.
- A draft task is not ready, so it never shows under Ready: drafts get a subsection of their own after the ready and layered tasks (built: `### Draft (n)` in the Tasks section, see `internal/boardengine/doc.go`).
- README.md is the root project's file: the projects under root first, each linked with its short description, then every task and note that belongs to no other project.
- No lead line reads as if every project were a beam: the Projects section's lead line, "Groups of entries; a beam spawns its members from its own worktree." (`projectsSection` in `internal/boardengine/render.go`), goes, since most projects are grouping only (from E5, operator 2026-10-25).

The board README rendering is not regarded as production code (operator, 2026-10-27).
