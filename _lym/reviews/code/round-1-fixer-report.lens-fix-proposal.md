```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposed fixes (F1–F5, all carried out as proposed; F5 also gained a `DeleteRemoteBranchLeased` row) for cases where the proposal itself is wrong or breaks something.

no findings

- **F1:** the row `older..head-1` answers true only for `IsAncestor(remoteAfter, startHead)`. Swapped arguments ask `head-1..older`, get false and end `pushed`, so the row catches the defect. A remote tip moving back to an ancestor ending `fixed` matches the documented contract of `run`: only commits the start head lacks, or a deletion, count.
- **F2:** setting `ancestorErr` affects no other row, because it is nil by default. The error path returns `%w`, so the `errors.Is` check holds. A nil `err` would make `errors.Is` false and short-circuit before `err.Error()`, so the assertion cannot panic.
- **F3:** both new rows assert the trailing `;`, which pins the exact session reference. With `keepRunDir` false and `runDir` empty, the check for `"run dir "` still holds for these reasons.
- **F4:** the added fragment matches the halt format at `finalize.go:1144`, and the gate passed after the spacing correction.
- **F5:** the delete rows seed origin with plain git, so the only `push landed` line for `refs/heads/feat` is the delete's. The log shows all three subtests passing.

Tool calls: none.
