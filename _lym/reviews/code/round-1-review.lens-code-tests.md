```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

Per-behaviour list:
- Baseline stencil Step 1 (stated output invariant wins, expect no drift, name the contradiction as a risk): no test. This is static stencil prose, which the lens exempts. No new assertion on this prose was added.
- Baseline stencil Step 2 `full` bullet (the verify agent judges each drifted test against the task and the expectation): no test. Static prose, exempt.
- Baseline stencil Step 4 (no acceptance reason or tolerance, state expected drift, name contradictions as risks): no test. Static prose, exempt.
- Verify stencil Step 2 (the report is expectation only, and drift under a stated invariant is a defect): no test. Static prose, exempt.
- Markers: the diff adds and removes no `{{.…}}` marker. `contracts/stencils/markers_test.go:66-67` pins the marker lists for both stencils and stays valid, so a marker regression would fail there.
- Rendering: `BaselineTable` and `VerifyTable` fill the same markers as before. `internal/driftengine/baseline_test.go` drives the producer over fakes and does not assert stencil prose. Nothing the diff changes alters Go behaviour, so no uncovered code behaviour exists.

Findings: none. The diff changes only static stencil prose and adds no test. I saw no assertion on stencil prose added, and no code behaviour without coverage.

Tool calls: none. The manifest already held the diff, `markers_test.go` and `baseline_test.go` lines, so I made no calls.
