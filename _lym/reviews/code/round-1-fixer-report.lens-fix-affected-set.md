```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: lens fix-affected-set, rerunning the search behind each finding.

- **F1** (recipes whose Drift expect agent gets a plan directory): `internal/loomrecipe/doc.go` lists the Drift rows for darn (line 27), sampler (line 36) and loom/tapestry (line 22). That matches the three recipes in `TestWire_DriftExpectPlanReading`. Pawl is not in that list and has no Drift-Expect row.
- **F2** (docs that state the Drift-Expect ordering or the `drift.enabled` entry gate): `docs/overview.md:1014` already says the sampler rows carry `drift.scope == full`, and the edited lines 1226 and 1231 now agree with it. The only other hit is `docs/research/drift/drift-whole-review-2026-11-04.md:36`. It is a dated research report, not text the fix had to update.

Calls:
1. Grep of `Drift-Expect` in `internal/loomrecipe` (non-test Go) and of the entry-gate and ordering phrases across docs, to cover the F1 recipe set and the F2 doc sites in one call. My context held neither.
