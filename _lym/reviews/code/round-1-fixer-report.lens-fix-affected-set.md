```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's own search and check that the fix covers every member.

**Findings left unchecked:** none. I checked both F1 and F2.

**F1 (ending-only differences):** no findings.
- `gofmtFinding` is the only formatting check built on the line loop. `fmtverb.go` compares bytes, so `lym loom fmt` does rewrite these cases. `tools/tapestryrename` formats output and is not a check.
- Both named cases, a missing final newline and extra trailing blank lines, have rows. Any other trailing difference, such as whitespace on a last line, still reaches the per-line comparison.

**F2 (claims about how far the walk reaches):** one finding.
- **LOW.** Changed text: `_lym/landing/summary.md`, the paragraph ending at line 9, "so each task's tier0 run fails on a Go file gofmt would rewrite."
  - **Summary:** this sentence still claims the guard's reach without the testdata exemption that the fix added at lines 3 and 20.
  - **Evidence:** gofmt would rewrite a mis-formatted `.go` file under a testdata directory, but `skipDir` (`strings.Contains(name, "testdata")` in `scankit.go`) means the guard never reads it.
- The other reach claims carry the exemption: the `gofmt_test.go` header and test doc comment, `doc.go`, and summary lines 3 and 20. `fmtverb.go`'s help text makes no "every file" claim.
- The earlier commit message still says "every Go file", but it is history and out of reach.

**Tool calls (2 of 6):**
- `git grep` for `format.Source|gofmt -l|gofmtFinding`: my context had no module-wide list of formatting-check sites for F1.
- `git grep` and `grep` for reach claims, including `summary.md`: my context had no module-wide list of places repeating F2's claim, or the summary in its changed form.
