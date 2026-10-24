# Board README headings show how many entries each section holds

Every section heading the rendered board README carries the number of entries listed under it in parentheses, such as `## Tasks (10)`, `### Ready (0)` and `### Enhancements (34)`.
The number is the count of entries in the section, not the highest handle number, since handles have gaps.

## What changed

- `renderReadme` in `internal/boardengine/render.go` writes every heading through a new `countedHeading` helper: Tasks, Running, Ready, each dependency layer, Independent, Notes, each type subsection, Other and Done.
- The Tasks count is the open tasks across its subsections and the Notes count the open notes across its subsections, so a parent count equals the sum of its children.
- Ready is always rendered, so an empty one reads `Ready (0)` above its `_None._` line.
- The render test expectations in `internal/boardengine/render_test.go` carry the new heading form, and the package doc in `internal/boardengine/doc.go` describes it.

## Design decisions

- `readmeSection.name` stays the bare name; the count is appended where the heading is written, so a section name is usable where no count applies.
- Done gets a count as well, since the change asks for every rendered heading.
