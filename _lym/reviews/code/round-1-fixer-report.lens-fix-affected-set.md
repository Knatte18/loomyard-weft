```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none.

The new row covers both `noticeFits` call sites that decide a refusal.
- **Line 667, the pre-record check:** The row walks it. A one-entry add with a failing resolver is recorded and committed, so `noticeFits` returning true on error is pinned.
- **Line 344, `notifyLiveRuns`:** It runs only when a batch has more than one entry, so it is not in the row's path. It calls the same `noticeFits`, so its error branch is the one the row pins. No review finding names it, and the rubric does not count that as a finding.

The production resolver at line 787 is the only `programName` site outside the fake. The fake at `decision_test.go:142` now returns `f.programNameErr`.

Calls made:
1. `grep -rn "programName\|noticeFits"` over `internal` plus `sed` of `decision.go` lines 660-672 and 780-795: to list every site under F1's key and read the two call sites and the production wiring, which my context did not show.
