---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/render_test.go:809
    summary: No README row pins a pipe in a project title, which the summary now emits unescaped
    fix: In the README golden row for a board with no prime owner, add a pipe to a project title, for example "Loose & <odd> | pipe". Expect it unescaped in the summary, "Loose &amp; &lt;odd&gt; | pipe". Add no new test function.
---

### [LOW] No README row pins a pipe in a project title

**Location:** `internal/boardengine/render_test.go:809`, behaviour at `internal/boardengine/render.go:526`.
**Issue:** `collapsibleProjectSection` replaced `cellText`, which escaped a pipe as `\|`, with `html.EscapeString`, which leaves a pipe alone.
That is correct, since HTML shows it as a pipe, but no golden row has a title with a pipe.
A later switch back to pipe escaping would show a backslash in the summary and fail no test.

## Rejected

- Markdown in a title (backticks, `*`) now shows literally in the summary, where the old heading link rendered it: inherent to the Goal's HTML `<a>`, and the old `cellText` path did not promise it.
- The README outline loses the `##` anchors for top-level sections: a direct consequence of the Goal's `<summary>`, not a defect of the diff.
- Lens code-correctness and code-affected-set returned no findings; the handler's own holistic read of the diff against the Goal found no scope gap or contradiction either.
  The nested sub-project sections stay on the old heading path, and only the top-level sections and the root's change.
