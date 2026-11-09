```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that pins it, or none.

**Behaviour → pinning test**

- **`&lt;details open&gt;` wrapper, with `&lt;summary&gt;` and a blank line after it:** `TestRenderReadmeGolden`. The `projectSummary` helper builds the exact opening string, and every README row compares it by full equality (`render_test.go`, helper near line 1243). Pinned.
- **Blank line before `&lt;/details&gt;`, and one between sections:** `TestRenderReadmeGolden`, via `readmeSectionEnd` (`"\n&lt;/details&gt;\n"`) followed by `"\n"` in the multi-project rows. Pinned.
- **Summary content (handle and title in an `&lt;a href&gt;` to the project file, `(N)` open count, `&lt;strong&gt;` wrapper):** `TestRenderReadmeGolden`, rows "each project renders to its own file…", "a project dependency shows by handle…" and the lone/loose row. Pinned.
- **Open-member count, including the root excluding its beside-it projects:** the "each project renders…" row has the root at `(1)` with `rootEntriesWithProjects(3)`. Zero counts are pinned by P6, P1 and P2. Pinned.
- **Root section is also collapsible and comes last:** `rootProjectSummary` is used in every README row, and the root is last in the multi-project rows. Pinned.
- **HTML escaping of a title (`&amp;`, `&lt;`, `&gt;`):** the lone/loose row uses `Loose &amp; &lt;odd&gt;` and expects `Loose &amp;amp; &amp;lt;odd&amp;gt;`. Pinned for `&amp;`, `&lt;` and `&gt;`. Handle goes through the same call, so its escaping is not separately pinned. That is acceptable.
- **Href escaping (`html.EscapeString(projectFile(...))`):** none. Project file names come from slugs, which are constrained, so I see no reachable input. I'm not raising it.
- **Nested sub-projects still render as `###` headings with markdown links:** `TestRenderReadmeGolden`, the "P1 One / P2 Two" row, still asserts `### [P2 Two](project-two.md) (1)`. Pinned.
- **Level-six clamp and project files unchanged:** the existing rows and the `files:` expectations are unchanged in the diff. Pinned by the existing tests.
- **`projectFile` refactor, `projectLink` and `openMembers` extraction, and the `projectSection` / `projectSectionContent` split:** no per-helper tests were added. They are covered through the public render golden. Test economy is fine, and I found no duplicate tests.

**Findings**

1. **LOW, design.** `render_test.go` (the lone/loose row, near lines 799–825), with the behaviour at `render.go:526`.
   - **Gap:** the summary no longer pipe-escapes, because `cellText` was replaced by `html.EscapeString`. It also collapses whitespace with `strings.Fields`, which `cellText` may have done differently. No row has a project title containing `|` or a newline or repeated spaces, so a regression in either would not fail any test.
   - **Evidence:** the only new odd-title fixture is `Loose &amp; &lt;odd&gt;`. The focus directive names `|` explicitly. Adding `|` to that title (expected `|` unescaped in the summary) would pin it in the same row, with no new test.

**Tool calls:** none. The manifest and diff already held everything needed.
