```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: no other test pins RunRecord.Failures; runTier is called only from internal/gatecli/run.go, with no fixture reaching tierrun.go:115.
- F2: not re-run; the finding names one missing row and it was added.
- F3: not re-run; the new row covers the one missing case.

Tool calls: 1. grep for Failures in other test files and runTier( call sites.
