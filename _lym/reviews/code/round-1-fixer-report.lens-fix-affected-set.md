```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F1's premise covers one function and one test. The grep shows `TestWorkspaceFolderNames` is the only test of `WorkspaceFolderNames`. The other sites are its definition in `handles.go`, the doc line in `doc.go` and the seam install in `cmd/lym/main.go`, and none of them needs a row. The fix covers the test's single table.

Unchecked: none.

Tool calls:
- `grep -rn "WorkspaceFolderNames" --include=*.go` over the worktree, to list every test and caller under that key, which my context lacked.
