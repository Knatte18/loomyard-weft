```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: the fix-affected-set lens, covering the tests and callers that each finding's premise puts under one key.

- **F1 (replayer migration of unversioned sides):** the search found three `Decode` calls, all in `replayer.go:71`, `:75` and `:79` for base, parent and commit. Outside the tests, `Decode` has no other caller. The new subtest feeds unversioned bytes to all three sides and asserts the result. A raw decode on any one side would change the `a` entry's targets, or leave `c` as `claude-file`. The set is covered.
- **F2 (`upgradeStore` dedup and `max` branches):** `upgradeStore` is called only from `Decode` and `Load`, and both are exercised by the extended table, which checks the `Load` and `Decode` results for each row. The dedup row `AgentsFileBesideClaudeFile` and the version 2 row cover the dedup branch and the `max` path. The Save round-trip assertion covers the `max` write path in `MarshalJSON`. The set is covered.

Findings left unchecked: none.

Tool calls:
1. `grep` for `Decode(` and `upgradeStore(` callers across `internal` and `cmd`. My context did not hold the call-site set outside the files I had read.
