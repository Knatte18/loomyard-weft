```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, rerunning each set-premised finding's search and checking the fix covers every member.

Findings left unchecked: none.

**no findings**

- **F1** (steps in parallel groups that take over process-global state):
  - In `lifecycle_integration_test.go`, the only matches are `InstallWorktreeSeam` at :267, in a top-level test that is already serial, and `logcapture.Capture` at :1406, in the step that moved.
  - The other `logcapture`, `Setenv` and `Chdir` matches in the package sit in separate test functions, not in a rows step.
  - Outside `docs/research`, the moved step's name appears only at the new test and the step definition, so no table row, pattern entry or subtest path is left behind.
- **F2** (steps that leave the hub changed): the review already lists the three members that run mid-group, and it says no later step trips on them. The rewritten sentence allows for exactly that. I did not search beyond the review's list, because this premise has no textual key to search on.
- **F3** (a single site, `summary.md` under Measurements): no set to cover.

Tool calls:
1. `grep` over `internal/battencli/*_test.go` for process-global state touchers: my context held no package-wide search for them.
2. `git grep` for the moved step's name outside `docs/research`: the earlier search keyed on `TestBattenIntegration_Rows`, not on the step's name.
