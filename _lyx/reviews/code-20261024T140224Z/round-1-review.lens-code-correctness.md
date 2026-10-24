```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Result: no findings.**

The diff's Go changes are comments, test failure-message strings and two new `doc.go` files. It adds no function, branch or error path. This is the per-change list, each case judged from the diff text:

- **Test failure messages:** the re-pointed `PATTERN-…` IDs and package-doc references in about 20 test files. Each is a `t.Errorf` or `findings = append(…)` format string with its arguments unchanged, so every `%s`, `%q` and `%d` verb still matches its arguments. Correct.
  - The `agentname` message now reads `"role-constant rule violated (see agentname's package doc): "+rel+…`, which still concatenates correctly.
  - The three `fabricengine` mutation-record messages keep their verb/argument counts.
- **`tools/sandbox/pathresolve_guard_test.go`:** the header comment and the `t.Errorf` string changed. The assertion logic is untouched, and the new message keeps `%s` with `strings.Join(failures, "\n")`. Correct.
- **New `internal/configreg/doc.go`:** it holds one package doc comment, preceded by a file-level comment and a blank line. The old `configreg.go` has no `// Package configreg` comment and no `// Sandbox` comment (checked by `git show` and `awk`), so there is no duplicate or conflicting package doc. Correct.
- **New `tools/sandbox/doc.go`:** it also carries a single `// Sandbox clones…` package doc ending in `package main`, with a file-level comment before it. `report.go:16`'s "Sandbox report file naming" is an ordinary comment, not a package doc. Correct.
- **Other `doc.go` edits:** these are comment-only, so they cannot affect behavior.

I did not run a build, so I have not confirmed the two new files compile. They are comment plus `package` line only, and there is no code in them to break.

Tool calls (3):
1. `git diff … internal/configreg/doc.go` plus a grep for `package configreg` and `package main`: I needed to see whether the new `doc.go` files were whole files, since my context only showed `+package` lines.
2. `awk` over `internal/configreg/*.go` and `tools/sandbox/*.go` for package-doc openers, plus `git show ed98ccf71:internal/configreg/configreg.go`: I needed to rule out a second package doc comment in the same package.
3. The first two calls ran with no further reads.
