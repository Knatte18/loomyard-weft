```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `_lym/landing/summary.md` edits for agreement with their surroundings.

No findings.

- Line 26 now says "an import the rule below does not allow". The design bullet below states that rule: added imports allowed when own, linked or standard library, and removed imports allowed when own or standard library. This matches `packageChanges` (`testselect.go:298-309`), `importAllowed` (`testselect.go:490`) and `standardLibrary` (`testselect.go:499`).
- The `MissTestFile` reason text (`testselect.go:82`, "test file changed outside its Test functions") still holds.
- Line 28 ("A linked package's test files are still skipped") is unchanged and still agrees with the code at `testselect.go:291`.
- No other file in the repo restates the old "no linked-package allowance" claim. The only other `MissTestFile` sites are in `declcompare.go`, and those cover package clause and non-Test declarations, not imports.

Tool calls:
1. Grep across the repo for `MissTestFile`, `importAllowed` and `standardLibrary`. My context held `packageChanges` but not the reason constant, the helpers' bodies or any other mention of the claim.
