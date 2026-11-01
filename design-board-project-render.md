# One project render for the board README and project files

`board-project-render` · Task · enhancement, board · running · Darn-Write

## Seen (operator, 2026-11-01)

The board README shows each open project as one flat member table, so a running member of a project does not show as running, and its ready, layered and draft tasks and its notes by type are not told apart.
Only the root's own entries, under the README's Prime section, and a project's own file get the full Tasks (Running, Ready, layers, Independent, Draft), Notes by type label and Done split of `entrySections`.
A project file lists the projects under it only as bullet links (`projectsSection`), and the README renders projects through a third path (`projectSections`).

## Design (operator, 2026-11-01)

- The root is a project like any other, Prime, rendered the same way as every other project.
- One shared project render function in `internal/boardengine` turns a project into markdown at a given heading level: its heading and facts, its brief, then the projects directly under it, each rendered by the same function one level down, then its own Tasks, Notes and Done exactly as `entrySections` splits them today.
- README.md is that function applied to Prime: its sub-projects first, each with Running, Ready, Tasks, Notes by type and Done of its own, then Prime's own entries at the bottom.
- A project's own file shows only that project and the projects under it, through the same function.
- `projectSections`' flat member table and `projectsSection`'s bullet list go; the file header (`fileHeader`) and the intro stay.
- Heading levels nest with the project depth and stop at level six, so a deep project still renders.
- The render package's doc comments say what the files now hold.
