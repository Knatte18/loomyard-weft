# Render every board project, Prime included, through one project section

The board README showed each open project as one flat member table, so a running member did not show as running, and its ready, layered and draft tasks and its notes by type were not told apart.
Only the root's own entries and a project's own file got the full Tasks (Running, Ready, layers, Independent, Draft), Notes by type and Done split.
A project file listed its sub-projects only as bullet links, and the README rendered projects through a third path.

Now `internal/boardengine` has one project render, and the root is a project like any other, Prime:

- `projectSection` renders a project at a heading level: its heading, facts line and brief, then `projectBody`.
- `projectBody` renders the open projects directly under it, each through `projectSection` one level down, then the project's own Tasks, Notes and Done through `entrySections`.
- `README.md` is `projectSection` applied to Prime, so every open project shows its own Running, Ready, Tasks, Notes and Done, and the root's own entries come last.
- A project file keeps its header as the project's heading, then shows its sub-projects and own entries through `projectBody`.
- `projectSections` (the flat member table) and `projectsSection` (the bullet list) are removed; `fileHeader` and the intro stay.
- Heading levels nest with project depth and stop at six, so a deep project still renders.
  `entrySections` and `countedHeading` now take an integer level, and a new `heading` helper applies the cap.
- The doc comments in `internal/boardengine/doc.go` and `render.go`, and the board paragraph in `docs/overview.md`, describe what the files now hold.
  The Prio column now belongs to Notes tables only, since the project member table is gone.

## Design decisions

- An open project under a done project still counts as under the root, since a done project has no section.
- A non-root project's heading keeps its link to the project file and its open-member count; Prime keeps its meaning line.

## To check

- The board entry's `Added after Discussion` decision says a project with a body should link its slug in the facts line to `design-<slug>.md`, the way `slugCell` links a task's slug.
  This diff does not do that: `projectFacts` is unchanged, so a project's body (for example P2 agent-turns' wave order) is still not reachable from the rendered board.
  Decide whether to land that here or follow up.
- Run `lym board` render on a real board with nested projects and read the README and a project file, since the new goldens in `render_test.go` cover synthetic boards only.
