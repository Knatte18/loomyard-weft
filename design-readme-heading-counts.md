# Board README headings show how many entries each section holds

`readme-heading-counts` · Task · enhancement, board · running · Finalize

## Seen (2026-10-24)

The rendered board README (`internal/boardengine/render.go`) numbers entries by handle (E1, E2, ...), but handles have gaps, so no heading tells how many entries a section holds.

## Change (operator, 2026-10-24)

- Every section heading the README renders carries the number of entries under it in parentheses: `## Tasks (10)`, `### Running (6)`, `### Ready (1)`, each layer heading, `## Notes (62)`, `### Audits (7)`, `### Enhancements (34)`, the `Other` notes heading too.
- The number is the count of entries listed in that section, never the highest handle number.
- Update the render tests and the package doc for the new heading form.