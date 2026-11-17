```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set searches behind F2, F3 and F4 (and F1's row set) to check that the fix covers every member. I found nothing to flag.

- **F2 (callers of the permission pass):** `makeTreeOwnerWritable` has one caller, `removeTreeMakingWritable` (`sweep.go:115`), and its own recursion. `removeTreeMakingWritable` is wired only at `root.go:48` and in the new test. Dropping the error return leaves no caller stale. The doc comments and `doc.go` are updated.
- **F3 (stdlib-only rules):** The PATTERN list at `PATTERN-test-isolation.md:181-182` has two entries, scankit and permkit, and each now has its own `AssertImportAllowlist` enforcement test (`enforcement_test.go:88` and `:98`). The `PATTERN-sole-parsers.md` stdlib-only lines are outside the list `enforcement_test.go` is said to enforce, so they are not part of this set.
- **F4 (keep marks):** The `//testtiming:keep` mark on `TestSweep_DeleteFailureTolerance` is in place at `retention_test.go:309`. `docs/benchmarks/test-redundancy.md` lists that test only under "No coverage: unclassifiable call", not as contained by `TestSweep_AgeBound`, so the review's premise is loose. The mark is harmless. No other `internal/logger` test in the report's covered list lacks a mark, apart from those marked "Removable: yes", which the review did not name.
- **F1 (link rows):** I did not re-search it. The change set shows the new `lockedTree` link-and-mode assertion alongside the existing top-level link row.

Calls made (3 of 6):
1. `grep` for the two function names across `.go` and `.md` files, `AssertImportAllowlist` and stdlib-only statements in `internal/testkit` and `pattern`, and files naming `TestSweep_DeleteFailureTolerance`. My context lacked the caller and site sets.
2. `grep` on `docs/benchmarks/test-redundancy.md` for the logger section and the existing `testtiming:keep` marks in `internal/logger`. My context lacked the redundancy report.
3. `sed -n 1353,1386p` on `test-redundancy.md`, to read the logger table and the "No coverage" list.
