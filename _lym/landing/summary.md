# A project page shows the project's body under its brief

`project-<slug>.md` showed a project's title, facts, brief and member lists, but its body lived only in `design-<slug>.md`, so reading a project's goal took a click through the slug link.

## What changes

- `internal/boardengine/render.go`: `fileHeader` now appends the project's body after the brief and the beam marker, before the intro paragraph and the sub-project and entry sections.
  A project with no body, or a whitespace-only one, renders as before.
- The body's ATX headings shift down together until the shallowest sits at level two (`bodyTopHeadingLevel`), capped at level six, so the page keeps its one level-one heading.
  A body whose headings already start at `##` or deeper is left as written.
  Lines inside fenced code blocks (backtick or tilde, closed per CommonMark) and indented code are never read as headings, either for the shift amount or for rewriting.
- `design-<slug>.md` and the facts line's link to it are unchanged, and README.md's project sections stay without the body.
- `internal/boardengine/doc.go` and the `render.go` file header describe the new project-file opening.

## Design decisions

- **Shift to level two rather than a fixed demotion by one.**
  A fixed `+1` would turn a body that opens at `##` into `###` directly under the page's `#`, skipping a level.
  Shifting so the shallowest heading becomes `##` never adds a second `#` and never skips a level; the body's sections then sit beside the page's own `##` sections (Tasks, Notes, sub-projects), which follow it.
- **No wrapping heading over the body.**
  The record asks only for the body under the brief; adding a "Body" heading would push every body section down a further level for no gain in the outline.
- **Setext headings (`===`/`---` underlines) are not shifted.**
  A `---` line is ambiguous with a thematic break, and board bodies use ATX headings.

## Tests

`TestRenderProjectFileShowsTheBodyUnderTheBrief` in `internal/boardengine/render_test.go` pins the file's opening for no body, a blank body, a plain body, a top-level-heading body that shifts, a `##` body that stays, code lines that stay, and a fenced `#` line that does not force a shift.
The existing project-file golden gains the `ship` project's body line.
