# A project slug on the board links to its design file

`project-design-link` · Task · enhancement, board · running · PR-Review

## Seen (operator, 2026-11-01)

A project's body, such as `agent-turns`' waves, renders to `design-<slug>.md`, but nothing on the rendered board links to it, so it cannot be read there.
`board-project-render` and `board-no-project-heading` both landed without the link.

## Design (operator and orch, 2026-11-01)

- In a project's facts line, in `README.md` and in project files alike, the project's slug links to its `design-<slug>.md` when the project has a body, the way `slugCell` links a task's or note's slug.
- A project without a body keeps the plain slug.
