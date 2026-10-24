```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (the `slices.Sort` swap in `scope.go`, the new carry row, the added assertions in the failing-vet row) for agreement with its surroundings.

no findings

- `scope.go`: the earlier `grep -n "sort\."` came back empty after the swap, so no use of the `sort` package is left behind. `slices` was already imported, so the import list is consistent.
- New row, `--card carries a vet run that passed on the same clean tree`: it sits in the section the comment describes as rows that commit to the pair's branch and run in order.
  - It removes `greenPath` on entry, as its sibling rows do.
  - It restores HEAD with a cleanup `reset --hard` to the revision it started from, so later rows see the same branch history as before.
  - It uses `vetSummary` and `tier1File`, which are defined earlier in the same function and compiled.
  - It does not use `wantCarried`, which asserts the `all` scope form, and the comment says why.
  - Its `" carried "` count of 3 (listed tier, vet run, guard step) matches the carried output captured in the first failed run.
- Failing-vet row: the added assertions use `runRecordAt` and `guardRunRecordOf`, which exist in `test_integration_test.go`. The expected `TierList` of `[0]` agrees with `tierNumbers`, which leaves out vet and guard runs.
- Docs: nothing in `doc.go` or `docs/overview.md` describes the changed lines, so no twin needs updating.

Tool calls made: none; my context already held every changed file.
