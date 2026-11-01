# Render every board project, Prime included, through one project section

The board README showed each open project as one flat member table, so a running member of a project never showed as running, and its ready, layered and draft tasks and its notes by type were not told apart.
Only the root's own entries under Prime and a project's own file got the full Tasks, Notes and Done split.

## What changes

`internal/boardengine/render.go` now renders every project through one recursive pair, `projectSection` and `projectBody`:

- `projectSection(project, level)` writes the project's heading, then its body.
  For the root the heading is `Prime` with its meaning line;
  for any other project it links the handle and title to the project's file and counts its open members, followed by the facts line and brief.
- `projectBody(project, level)` writes a section per open project directly under it, one heading level down and in `childProjects`' order, then the project's own entries through `entrySections`, one level down.

README.md is the file header, the intro, then `projectSection` of Prime at level two: Prime's sub-projects first, nested to any depth, each with its own Running, Ready, layers, Draft, Notes by type and Done, then Prime's own entries last.
A project file is its header, the intro, then `projectBody` of its project at level one, so its sub-projects and its own Tasks, Notes and Done sit at `##`, as the own entries did before.
`projectSections` (the README's flat member tables) and `projectsSection` (a project file's bullet list) are removed, along with the `projectsHeading` constant.

Headings take an integer level through a new `heading` helper capped at `maxHeadingLevel` (six), so a deeply nested project still renders; `countedHeading` and `entrySections` take that integer level instead of a `#` string.

## Design decisions

- A project file calls `projectBody`, not `projectSection`: its file header already carries the project's H1, facts line and brief, so rendering the section heading again would repeat them.
- A sub-project's heading keeps the open-member count it carried before, matching the rule that every section heading but Prime ends in a count.

## Docs and tests

The `# README` section of the `boardengine` package doc, the `render.go` file header and the board paragraph of `docs/overview.md` describe the nested layout.
`TestRenderReadmeGolden` pins the nested README and project files for the existing project fixtures and adds a row with a chain of four nested projects whose headings stop at level six.
