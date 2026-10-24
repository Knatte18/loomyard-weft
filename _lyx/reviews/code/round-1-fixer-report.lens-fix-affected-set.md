```yaml
lens: fix-affected-set
outcome: reported
```

Findings: one LOW.

**LOW** — `internal/websterengine/recoverbatch_test.go:1590-1612`, the zero-count rows. The branch sits behind four checks: kind, terminal, dead status and cap. The fix pins kind and terminal, as the review asked. The dead-status check has no zero-count row, because the stuck and failed rows all use `deadRecovery(1, start)`. Moving `if recoveries == 0` above the `bs.Status != DigestStatusDead` check would still pass every row. A stuck or failed record that a reset cleared would then retry. The review said existing rows pin dead status, but only for a count of 1, so that claim doesn't cover the zero-count branch. The cap check can't be reordered against the zero branch, since a count of 0 never reaches the cap. A zero-count row with `Status = DigestStatusStuck` (or Failed) would close the gap.

The review asked for the kind and terminal rows only, so this is an extension of F1's own premise, not a defect the fix introduced.

Tool calls:
1. `grep -n "deadRecovery(0"` over the package's tests, to list every zero-count row. My context showed only the file I'd read, so I wanted to rule out zero-count rows elsewhere in the package.
