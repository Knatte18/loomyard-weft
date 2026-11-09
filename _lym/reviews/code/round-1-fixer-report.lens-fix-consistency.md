```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (pipe added to the "Loose" project title in the README golden row) for consistency with its surroundings.

No findings.

- The task title at `render_test.go:802` and the expected summary at `render_test.go:818` both carry `| pipe`. The summary text is the escaped form of the title, and `project-loose.md` comes from the slug, so the pipe does not change it.
- The row has no `files` entry for `project-loose.md`, so no golden project-file text needed the new title. It lists only `project-lone.md`.
- A grep of `internal/` Go files for the old title and its escaped form finds only these two lines, so no twin of the change was left behind.
- `collapsibleProjectSection` runs the title through `html.EscapeString`, which leaves `|` alone, so the expectation matches `render.go:526`.

Tool calls:
1. `grep -rn "Loose &amp;\|odd&amp;gt;\|&lt;odd&gt;" internal/ --include=*.go`: to check that no other test or golden file uses the old title. My context lacked a search across the package.
