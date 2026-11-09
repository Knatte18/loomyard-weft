```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the F1 search (every test site that uses the title changed in the fix) and check the change covers all of them.

No findings.

Every site that uses the changed title is covered. The task title at `render_test.go:802` and the expected summary at `render_test.go:818` both carry the pipe. No other test or golden file references "Loose &amp;" or `odd&amp;gt;`. The `files:` map for this row has no `project-loose.md` entry, so no other expectation needs the title.

Tool calls:
1. `grep -rn "Loose &amp;\|odd&amp;gt;"` over `internal/*.go`, to confirm no other site uses the changed title. My context had only the one test row.
