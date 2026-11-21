```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The review's set is the outcomes of root resolution in `rootProcesses`, and the fix covers each one.

- **Resolved root beside the written one:** the existing process 12, with `Cwd` under the resolved root, is ended.
- **Written root kept after resolution:** the new process 13 names `filepath.Join(root, "hub-one")` in `Argv` with `Cwd` outside both roots. The test expects `[]int{12, 13}`.
- **Resolution fails:** the new subtest `EndsAnOrphanUnderTheWrittenRootWhenTheRootDoesNotResolve` uses a root that does not exist. It ends the process under the written root and leaves the `/elsewhere` one.
- **`resolved == deps.Root`:** the other `TestReap` rows build an existing root with no link, so they cover it.
- **Negative case:** process 14 (`/elsewhere`) in the link subtest is left alone.

`rootProcesses` has one caller, `reap.go:128`, and `EvalSymlinks` appears only at `reap.go:199` and in the test setup, so no other site needs a row.

Tool calls:
1. grep for `rootProcesses` and `EvalSymlinks` in `internal/hardenshed`, because my context held no search of the other call sites.
