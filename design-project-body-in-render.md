# A project page shows the project body under its brief

`project-body-in-render` · Task · enhancement, board · running · Code-Bouncer

## Seen (operator, 2026-11-03)

The Bolt page `project-<slug>.md` shows a project's title, brief and member lists; its body sits only in `design-<slug>.md`, linked from the slug, so the operator must click through to read the project's goal and design.

## Expected

- `project-<slug>.md` renders the project's body under its brief, before the task and note lists, with its headings demoted below the page's own so the page outline stays intact.
- A project with no body renders as today.
- `design-<slug>.md` and its link stay as they are.
