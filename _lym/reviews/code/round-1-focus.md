---
round: 1
focus:
  - "internal/boardengine/render.go `projectSections`: settle whether the README shows one section per open project, nested ones depth-first, with members that are open only, and whether the member table's order and columns match the Notes type-group layout the board entry asks for."
  - "internal/boardengine/render.go `renderReadme` and `entrySections`: settle whether the Prime section sits below every project section, and whether its Tasks, Notes and Done headings, and their subsections, drop exactly one level in the README while project pages keep their levels."
  - "internal/boardengine/render.go `table`, `memberTable` and the `columnPriority` constant: settle whether every table carrying the priority column is headed `Prio` and no stray `P` header remains, and whether the helper extraction from `renderBoardFile` changed any row, header or trailing blank-line output."
  - "internal/boardengine/render.go `fileHeader`: settle whether a non-beam project page loses the `grouping` line while a beam page keeps `beam` and its owner, and whether no other reference to the removed constant remains."
  - "internal/boardengine/render_test.go: settle whether the golden rows cover root-level and nested projects, an empty project showing `_None._`, a project with a brief, and a beam page, by extending the golden test rather than adding tests."
  - "internal/boardengine/doc.go, docs/overview.md and the render.go header comment: settle whether the README description matches the new layout, and whether any other document or file naming the old README layout, the `P` column or the `grouping` line was left unchanged."
  - "_lym/landing/summary.md: check each claim against the diff."
---

Seed focus for round 1. The change is confined to the board README rendering, its goldens and two docs.
