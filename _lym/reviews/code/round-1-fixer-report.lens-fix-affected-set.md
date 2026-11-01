```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: lens fix-affected-set. I reran the search behind each set-premise finding and checked that the fix covers every member.

- **F1 (parse failure aborts the report):**
  - `findLanding` has one call site, `tools/tokencount/main.go:334`, and it only propagates read errors now.
  - The only `TestFindLanding` row that expected the parse error is converted to `parseFails`.
  - No other test or doc pins the old error.
- **F3 (root commit):**
  - `ChangedLines` is declared on `landingRepo` and implemented by `codeRepo` and `fakeLandingRepo`.
  - It is called only from `landed.go:107`.
  - The new `TestCodeRepoChangedLines` covers root and non-root commits.
- **F4 (`-last` below 1):**
  - `recentRuns` is called from `main.go:289` and `count_test.go:496` and `:516`.
  - The `main.go` caller only passes `limit = 0` when `-since` is set, which the early refusal guarantees.
  - The `count_test.go` calls pass explicit `last` values, so they are unaffected.
- **F5 (wrapper headers):** `tokencount.sh` and `tokencount.cmd` were both edited, and the two wordings match.
- **F2 and F6** are test-coverage findings with no set premise, so I did not search for them.

Findings left unchecked: none.

Tool calls:
1. Grep for `recentRuns(` and `-last` across Go, md, sh and cmd files, to cover the callers and docs of F4.
2. Grep for `ChangedLines|landingRepo|findLanding|noDescription|diff --numstat`, to cover F1 and F3. The context showed the edited files but not the repo-wide references.
