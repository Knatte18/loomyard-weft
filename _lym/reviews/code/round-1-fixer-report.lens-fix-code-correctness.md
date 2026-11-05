```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 fix through the fix-code-correctness lens.

The fix changed no code. The only Go change is a comment in `internal/pattern/doc.go`, which adds two lines after the `(test)` definition. The other changes are one line, "Held by review alone; no test enforces it.", added to markdown background files.

- **`doc.go`:** No function, signature or branch changed, so there is no input to run and no caller to break. The new comment lines follow the one-sentence-per-line style and sit inside the existing package doc.
- **Background files and `Check`:** `Check` enforces the overview's shape and that each file under `pattern/` is named by exactly one entry. It does not parse background file bodies. The added lines are plain paragraphs, so no check can be affected. `lym gate test ./internal/pattern` passed (70 targets, 0 failed), which agrees.
- **Tests:** The fix adds none. The finding is a doc and consistency gap with no gate that catches it, and the review itself says so, so no test is owed.
- **Placement:** In every edited file, each new line sits directly under a `Rule:` line of a member that has no `(test)` marker. I saw this in the reads for `PATTERN-pair-remote`, `PATTERN-reed-panes`, `PATTERN-daemon-wakeups`, `PATTERN-stencil-ownership` and `PATTERN-run-records`. The `PATTERN-stencil-ownership` edit for "Reading prompt or spec" sits before "Every producer prompt…", so it stays attached to its own Rule.
- **Coverage (outside my lens):** The fixer did not add the line to `PATTERN-shed-driver`, whose three Rule lines at 106, 110 and 114 lack `(test)` and whose entry carries `(test)` in `PATTERN.md`. The review did not name that file. The fix-affected-set lens covers it.

no findings

Tool calls made: none beyond the context I already held.
