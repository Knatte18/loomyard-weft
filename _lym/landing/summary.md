# Bolt README: collapsible project sections

Each project section in Bolt's `README.md` is now a `<details open>` block the reader can collapse.
This covers every top-level project and the root's own section, which stays last.
Sections are open by default, so the README reads as before until the operator folds one.
Project files and design files render as before.

The `<summary>` carries what the `##` heading carried: the project's handle and title linked to its file, and the open-member count.
Markdown does not render inside `<summary>`, so the link is an HTML `<a>`, and the title is HTML-escaped.
A blank line follows `<summary>` and precedes `</details>`, so the markdown inside renders.
The section's contents keep their heading levels below `##`.

## Design decisions

- `renderReadme` in `internal/boardengine/render.go` calls a new `collapsibleProjectSection`.
  `projectSection` is split into a heading and `projectSectionContent`, which both it and the new function use, so project files render unchanged.
  The open-member count and the project file name are factored into `openMembers` and `projectFile`.
- The summary text is wrapped in `<strong>` so it looks like the heading it replaces.
- The summary does not escape a pipe in a title, since it is not inside a table cell.
  The golden test pins this with a title holding `&`, `<`, `>` and `|`.
- `doc.go` and `docs/overview.md` describe the new README structure.

## To check by hand

Golden tests pin the generated markup, but nothing renders it.
Open a generated `README.md` in VS Code's built-in Markdown Preview and on GitHub, and confirm that:

- each section collapses and expands;
- the summary link opens the project file;
- the tables and headings inside a section render.
