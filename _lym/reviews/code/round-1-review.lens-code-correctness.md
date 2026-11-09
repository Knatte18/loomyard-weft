```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff, covering `collapsibleProjectSection`, `projectFile`, `openMembers`, `projectSectionContent`, `projectSection` and `projectLink`.

**List (function or branch, case run, verdict)**

- `collapsibleProjectSection`, ordinary project (handle `P1`, title `Ship it`, slug `ship`): emits `&lt;details open&gt;`, then `&lt;summary&gt;&lt;strong&gt;&lt;a href="project-ship.md"&gt;P1 Ship it&lt;/a&gt; (6)&lt;/strong&gt;&lt;/summary&gt;`, then a blank line. After that come the facts, the brief and the body, and `projectSectionContent` ends in a blank line before `&lt;/details&gt;`, `""`. Correct.
- `collapsibleProjectSection`, root (`rootProject`, slug empty): `projectFile` returns `rootProjectFile`, and `openMembers` excludes `KindProject` entries. Facts is `Owner …`, and `projectBody` calls `entrySections(…, 3)`. Correct, and identical to before apart from the heading.
- `collapsibleProjectSection`, title `Loose &amp; &lt;odd&gt;`: `html.EscapeString` gives `Loose &amp;amp; &amp;lt;odd&amp;gt;`. Correct.
- `collapsibleProjectSection`, title containing `|`: the old heading used `cellText`, which escaped it to `\|`, and the old link text rendered as `|`. The summary now emits a plain `|`, which HTML shows as `|`. Same visible text, correct. A `"` becomes `&amp;#34;` and shows as `"`, and `'` becomes `&amp;#39;`. Correct.
- `collapsibleProjectSection`, empty handle or title: `strings.Fields` collapses the whitespace and trims, so there is no leading space and no panic. Correct.
- `collapsibleProjectSection`, title with a newline or `\r\n`: `strings.Fields` splits on any whitespace, so the summary stays on one line. This is equivalent to `cellText`'s collapse. Correct.
- `collapsibleProjectSection`, `href` from `projectFile`: the slug is escaped with `html.EscapeString`. Slugs are restricted characters, so there is no URL-encoding concern. Correct.
- `collapsibleProjectSection`, nested sub-project: `projectBody(project, 2)` still calls `projectSection(child, 3)`, which renders the old `###` heading path. So the README's nested sections stay headings, as the commit message claims. Correct.
- `collapsibleProjectSection`, deep nesting: `readmeSectionLevel` is 2, so the level-capping behaviour in `heading` is unchanged for the contents. Correct.
- `collapsibleProjectSection`, `append([]string{...}, ...)` into a fresh slice, then `append(lines, "&lt;/details&gt;", "")`: no aliasing of shared slices. Correct.
- `renderReadme` and `joinLines`: the trailing `""` is trimmed by `TrimRight`, so the file ends with `&lt;/details&gt;\n`. A blank line separates consecutive sections because the contents end in a blank line. Correct.
- `projectSection` (nested): the heading is `countedHeading(level, projectLink(project), openMembers)` followed by `""` and the content, byte-identical to the old output (heading, blank, facts, blank, …). Correct.
- `projectLink` and `projectFile`: the slug-empty branch gives `rootProjectFile`, and otherwise `projectFileName(slug)`. Same as before. Correct.
- `openMembers`: the condition and the count are unchanged, moved intact. Correct.
- `projectSectionContent`: facts, blank, optional brief and blank, then the body. This matches the old tail of `projectSection`. Correct.
- Rendering in a markdown viewer: the blank line after `&lt;/summary&gt;` and the blank line before `&lt;/details&gt;` are both present, so the markdown inside renders. `&lt;details open&gt;` is also present. This is correct on the HTML-block rules; I did not run a viewer.

No findings.

Tool calls (2):
1. Read `render.go:228-247` for `heading` and `countedHeading`: the manifest's reads did not include their definitions.
2. Read `render.go:796-815` for `cellText`: needed to compare its escaping with `html.EscapeString`.
