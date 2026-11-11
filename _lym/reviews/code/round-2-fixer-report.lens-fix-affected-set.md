```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The only text outside the changed summary that names `MissTestFile` is code. The three `declcompare.go` sites and `testselect.go:82,300,306` match the new wording: the package-clause miss, non-Test declaration changes, and the added and removed import rules. The two edited passages in `_lym/landing/summary.md` (lines 26 and 55–59) are the only prose descriptions of test-file import handling. The one other test-file import mention outside `docs/research/` is a runproc leaf-enforcement finding unrelated to the selector, and I did not read the other hits in the truncated grep output.

Unchecked: none.

Tool calls:
1. Grep for test-file import prose and `MissTestFile` across `_lym/landing`, `docs` and `internal/testcover`. I needed it to find sibling descriptions of the rule that my context lacked. The output was truncated.
2. Grep for `MissTestFile` outside tests and `docs/research`, to list the code and doc sites that state the rule. My context only held `testselect.go:230–410`.
