```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency check of the F1/F2 change set (commit `3f3d016c6` plus the `_lym/landing/summary.md` edits).

no findings

- The new ending-only finding agrees with `lym loom fmt`: `planGoFormat` marks a file changed on any byte difference, so the hint's command rewrites it. The help text in `internal/loomcli/fmtverb.go` still holds.
- The testdata exemption now reads the same in the `gofmt_test.go` header, the test's doc comment, `internal/testkit/doc.go` and `summary.md`.
- The search turned up no other statement of the guard's reach or of its "first differing line" message left stale.
- The older commit message of `19c8c6847` still says "every Go file of the module". It is history, outside this change set.

Tool calls:
1. `git grep` for claims about the gofmt guard: failed on pathspec syntax and read nothing. My context lacked a repo-wide search for twins in wording.
2. The same search with corrected pathspecs: my context's earlier grep matched only the guard's identifiers and file name, not prose claims about its reach.
