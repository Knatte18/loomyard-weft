```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions, branches and error paths.

**List: functions and branches the diff adds or changes**

- None. The diff changes no function, branch or error path. The parent context already checked this mechanically over `3d3b4812..HEAD`:
  - Every changed file is a `doc.go`: the filter for names not ending in `/doc.go` returned nothing.
  - Every added or removed line in `git diff -U0` is a `//` comment line or a blank line: the filter for non-comment changed lines returned nothing.
  - In every changed file, the line directly above `package` is still a `//` comment, so the package doc comment stays attached: the awk check flagged no file.
  - Every changed file still has a `// Package <name>` line: the check reported none missing.
  - The diff adds or removes no `//go:build` line, so no file's build constraints changed.
  - The reach report agrees: "the diff changes no function of the module". The mutation report ran 0 mutants.
- Case run: none can be run, because comment-only lines have no input or execution path. What a lens can verify is whether each file still parses as a Go file with an attached package doc, and the checks above confirm that.

**Findings**

None.

**Tool calls**

None. The checks above were already in the inherited context.
