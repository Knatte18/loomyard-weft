## Goal

```markdown
## Seen (operator, 2026-11-01)

In the rendered `README.md`, the Prime section lists its projects (P1, P2) as `###` sections, then the root's own Tasks and Notes as `###` sections at the same level.
Nothing marks where P2 ends and the root's own entries begin, so the README reads as if Tasks and Notes belonged to the last project.

## Design (operator, 2026-11-01)

- When a project has open sub-projects, its own Tasks, Notes and Done sit under one heading of their own, at the level of its sub-projects' headings, after them, with the entry sections one level further down.
- For the root that heading is `No project`, with a meaning line saying these are the entries under no project.
- For any other project with sub-projects it is `Own entries`, with a meaning line saying these are the entries directly under the project and under none of its sub-projects.
- A project without sub-projects renders as today, its entry sections directly under its heading.
- The change sits in `projectBody` in `internal/boardengine/render.go`, so the README and every project file follow it alike.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-01): A project with a body links its slug to its design file

Decision: Taken over from board-project-render, which landed without it: in a project's facts line, in the README and in project files alike, the project's slug links to its design-<slug>.md when the project has a body, the way slugCell links a task's or note's slug; a project without a body keeps the plain slug.

Rationale: Operator, 2026-11-01: a project's body, such as agent-turns' waves, is not reachable anywhere on the rendered board; the change sits in the same render file as this task.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
