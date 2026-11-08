```yaml
lens: fix-code-correctness
outcome: reported
```

The integration test starts from a fresh pair and writes no `summary.md` before drafting, so the new existence check does not break it. The MEDIUM finding above is withdrawn. No other integration test calls the draft seam.

No findings.

Calls made:
1. `grep -rln` for draft callers plus `sed -n 150,186p internal/beamcli/deps_integration_test.go`. I needed to know whether tier-1 tests draft over a pre-existing description, and the integration fixtures were not in my context.
