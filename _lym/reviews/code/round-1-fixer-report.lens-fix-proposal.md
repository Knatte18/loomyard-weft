```yaml
lens: fix-proposal
outcome: reported
```

No findings.

I tried to build a case where the pipe breaks something, and none held.

- **The README summary:** the pipe in the title appears only inside `&lt;summary&gt;`, which is not a table cell, so an unescaped `|` is correct there. The row asserts exactly that (`"P2 Loose &amp;amp; &amp;lt;odd&amp;gt; | pipe"`).
- **Other places the title appears:** `render.go:375` (via `cellText`), `render.go:395` (the project-file heading, also via `cellText`) and `render.go:716` (via `cellText`) all escape the pipe. This row's `files` map checks only `project-lone.md`, and the gate passed (783 passed, 0 failed), so no unlisted output changed.
- **Does the row pin the old behaviour?** If someone switched the summary back to `cellText`, the summary would show `\|`, and this row would fail. The row therefore tests what the finding asked for.
- **Separate gap, not caused by the fix:** no row checks the pipe-escaped form in a table cell or heading for this project. The review did not ask for it, so it is not a finding.

Tool calls made:
1. Ran `grep` on `internal/boardengine/render.go` for `cellText` and `.Title`, to see where else a project title is rendered. My context showed only lines 505–545 of `render.go`.
