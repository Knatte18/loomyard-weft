```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change (the subtest rename in `internal/runproc/end_test.go`) for sibling and twin consistency.

No findings.

- The commit changes one string, `...PushesTestMapSlackRelay...` to `...PushesSlackRelay...`, on one line.
- `grep -rn "TestMap" internal/runproc/` returns nothing, so no twin reference to TestMap is left in the package, in the test, `end.go` or `doc.go`.
- The renamed string is a subtest name only. I saw no reference to it elsewhere in the context.

Tool calls:
1. `grep -rn "TestMap" internal/runproc/`: checked for leftover TestMap twins after the rename, which my context lacked.
