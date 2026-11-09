# Bolt README: every project section can be collapsed

`readme-collapsible-projects` · Task · enhancement, board · running · Drift-Await

## This task

In Bolt's README.md, which `renderReadme` in `internal/boardengine/render.go` builds, wrap each project's section in `<details open>` with a `<summary>`, so the operator can collapse it.
That covers each top-level project and the root's own section; the per-project files stay as they are.

- The summary carries what the section heading carries now: the handle and title linking to the project's file, and the open-member count. Since markdown does not render inside `<summary>`, the link is an HTML `<a>`.
- A blank line follows `<summary>` and precedes `</details>`, so the markdown inside renders.
- Sections open by default.
- It must render in VS Code's built-in Markdown Preview, which the operator reads the board in, and on GitHub.

## Limits

Only the README's project sections change; the tables, the project files and the design files render as now.