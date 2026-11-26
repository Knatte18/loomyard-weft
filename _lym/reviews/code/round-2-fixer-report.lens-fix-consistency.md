```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of commit `3f0ab187a`, comparing the `tools/machinecost/newbinary.go` changes against the text and code they touch.

no findings

I checked these and they still agree with the change:
- The package doc in `main.go` says folders are made only under `-dir` and the temp root, and are deleted on interrupt, SIGTERM or hangup.
- The decision record's line 32 and landing summary line 38 make the same claim about folders.
- `runProbe`'s "stops with an error once ctx is cancelled" still holds, because the error now arrives after the running build ends.
- The `measureNewBinaries` doc and `newbinary_test.go` are unaffected.
- A `git grep` found no reference to `machinecost` in tracked files outside `tools/machinecost`.

Tool calls:
1. `git grep` for `machinecost` outside the tool, plus `git show --stat HEAD`: my context had no repository-wide references.
2. `grep` of `_lym` records for build, folder and interrupt claims: my context did not hold the decision record.
3. `grep` of `_lym/landing/summary.md`: my context did not hold the landing summary.
