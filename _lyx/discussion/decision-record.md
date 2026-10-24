# Decision record — readme-heading-counts

## Goal

Every section heading the board README renders carries the count of entries listed under it, e.g. `## Tasks (10)`, `### Enhancements (34)`.

## Scope

- `internal/boardengine/render.go`: `renderReadme` writes each heading as `<name> (<count>)`.
- Headings covered: Tasks, Running, Ready, each dependency layer, Independent, Notes, each type subsection, Other, Done.
- `internal/boardengine/render_test.go` expectations and the `internal/boardengine/doc.go` README description follow.
- Out of scope: the board data, handles, design docs, the table columns.

## Decisions

- The count is `len` of the entries the section lists, never a handle number.
  Rationale: handles have gaps, which is the reason for the change.
- The Tasks count is the number of open tasks across Running, Ready, the layers and Independent; the Notes count is the number of open notes across all type subsections including Other.
  Rationale: a parent heading's count equals the sum of its subsections, so the figures reconcile.
- Ready, always rendered, reads `Ready (0)` when empty, beside its `_None._` line.
  Rationale: the heading rule applies to every rendered heading, and an empty section is a count of zero.
- Done carries its count too.
  Rationale: the board entry says every heading the README renders.
- A single helper in `render.go` formats a heading from a name and a count.
  Rationale: one site declares the form, as the file already declares section names alone.
- `readmeSection.name` stays the bare name; the count is appended at the heading write sites.
  Rationale: `layerSection` names and the name constants stay reusable elsewhere without a count.

## Constraints

- PATTERN applies; no new cross-cutting invariant arises.
- Test economy: extend the existing README render table rows rather than adding tests.
- The package doc describes the new heading form in the same commit.

## Auto-mode assumptions

- A darn run holds no interview; the board entry's text is the whole statement.
- The count format is a space and the number in parentheses after the heading name.

## Open risks

- Another module that parses README headings would break; a grep found none outside `internal/boardengine`.

## Acceptance criteria

- Each rendered heading ends in ` (<n>)`, n being the entries listed under it.
- Tasks equals the sum of Running, Ready, layers and Independent; Notes equals the sum of its subsections.
- An empty Ready renders `### Ready (0)`.
- Gaps in handles do not change a count.
