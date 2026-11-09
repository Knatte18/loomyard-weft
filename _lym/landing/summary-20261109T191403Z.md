# Bolt README: every project section can be collapsed

The board's `README.md` now renders each open top-level project, and the root's own section, as a `<details open>` block, so the operator can collapse a project while reading the board.
Sections are open by default.

```markdown
<details open>
<summary><strong><a href="project-ship.md">P1 Ship it</a> (6)</strong></summary>

…facts line, brief, sub-projects and Tasks, Notes and Done, as before…

</details>
```

## What changed

- `internal/boardengine/render.go`: `renderReadme` renders each section through the new `collapsibleProjectSection`.
  Its summary carries what the `##` heading carried: the handle and title linked to the project's file, and the open-member count.
  The link is an HTML `<a>`, since markdown does not render inside `<summary>`, and the title is HTML-escaped.
  A blank line follows `<summary>` and precedes `</details>`, so the markdown inside renders.
- `projectSection` is split into `openMembers` and `projectSectionContent`, which the collapsible section shares, and `projectFile` names a project's board file for both the markdown and the HTML link.
- `doc.go` and `docs/overview.md` describe the collapsible sections.

## Design decisions

- The summary replaces the `##` heading rather than wrapping an `<h2>`.
  A block heading inside `<summary>` drops below the disclosure arrow in VS Code's Markdown Preview, so the summary is bold inline text.
  The headings inside a section keep their levels (`###` for Tasks, Notes and Done, and for nested sub-projects), so the tables render as before.
- Only README's top-level sections collapse.
  Nested sub-project sections inside them, the project files and the design files render unchanged.

## Tests

`TestRenderReadmeGolden` pins the new README form in every row.
The golden helpers `projectSummary`, `rootProjectSummary` and `readmeSectionEnd` hold the details markup.
The row for a board with no prime owner now gives a project the title `Loose & <odd>`, which pins that the summary escapes it to `Loose &amp; &lt;odd&gt;`.
