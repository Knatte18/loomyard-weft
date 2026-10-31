```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the findings F1, F2 and F5 and check the fix covers every member.

No findings.

- **F1:** `shedrun.ReadLandedCommit` has one production caller, `internal/battencli/dobbyreads.go:174`, and the fix covers it. No other reader can abort on this error.
- **F2:** `recordLanding` has exactly two call sites, `internal/landingshed/finalize.go:291` and `:311`. The first-try site now has `TestFinalize_RecordsTheLandingCommitOnTheFirstTry` and the retry site has its existing row, so both are asserted.
- **F5:** `RecordLanding` has one production wiring, `internal/loomcli/landingdeps.go:200`, and the new loomcli test covers it. The `ClearNotifyOnLand` closure in `internal/beamcli/dobbydeps.go:161` is covered by the new beamcli test. I did not search for other wirings of either seam.
- **Left unchecked:** F3 and F4. Their premises are single-test coverage gaps, not sets, so I ran no search for them.

Tool calls:
1. `grep` for `ReadLandedCommit(`, `recordLanding(` and `RecordLanding` across non-test Go files, to list every reader, call site and wiring. My context had only the files the fixer opened, not the whole tree.
