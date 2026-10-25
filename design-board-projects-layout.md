# Decide the layout of projects in the board README

`board-projects-layout` · Note · enhancement, board

## Seen (operator, 2026-10-25)

Project entries (P handles) and their members render in the README's Projects section since dd11ab09f.
The section's lead line (`projectsSection` in `internal/boardengine/render.go`) reads "Groups of entries; a beam spawns its members from its own worktree.", which reads as if every project were a beam; a project needs no beam, and most are grouping only (operator, 2026-10-25).
After the first two projects, `workspaces` and `dotnet`, were made, the operator was not satisfied with that layout.

## Idea

Decide with the operator how projects and their members are shown on the board, then change the README rendering to match.
The layout is not yet discussed.