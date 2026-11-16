```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the F1–F5 change set for twins or siblings left out of step.

- **F1:** `internal/websterengine/runlevel.go:575` renders "...recorded fingerprint %s; the plan changed since state.json was created; %s". The new `PlanEditError` closure in `internal/websterengine/runplan.go:156` produces the same text, so the refusal reads the same on every verb. No test or other file pins "since the run state was created".
- **F2:** `export_test.go` binds `checkRecoveredSuspects` to `recoveredSuspectReasons`, so `suspect_test.go` now covers the production call at `recoverbatch.go:689`. The `fmt` import removed from `export_test.go` had no other use there. `recoverbatch.go` still uses `fmt` and `runplan` elsewhere.
- **F3, F4, F5:** no stale heading or cross-reference remains. Grepping for "No shared substrate" finds no other mention, and nothing else refers to the old helper names.
- **Doc wording:** the `runplan/doc.go` claim of two webster-named texts matches the `conflict.go` and `postbatch.go` sites the review named.

Tool calls: 1 (a grep for the old heading and the old mismatch phrasing; the context held the edits but not the other files that might twin them).
