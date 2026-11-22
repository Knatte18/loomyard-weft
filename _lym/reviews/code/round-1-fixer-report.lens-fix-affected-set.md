```yaml
lens: fix-affected-set
outcome: reported
```

No findings. Task: check that the F1, F2 and F5 fixes cover every site in their sets.

- **F1 and F5 (docs).** A repo-wide grep of `pattern`, `docs`, `internal/gateslot`, `internal/gatecli` and `cmd` for report-step wording like "after the tiers", "released their slots" or "runs the report step after" found no site still describing the sequential tail.
  - The three `PATTERN-test-speed.md` lines (225, 330, 345) now say the steps judge beside the last tier.
  - `internal/gateslot/doc.go` lines 45 and 79-80 are updated. `internal/gateslot/tiers.go:118`, `internal/gatecli/reports.go:2` and `internal/gatecli/doc.go:366` already match.
  - `docs/research/speed/testmap-use-measure-2026-11-21/report.md:55` is a captured measurement report, which the prose rules exempt.
  - `internal/gatecli/mutation.go:1` ("after the other report steps") concerns step order, not the tail, so it stays true.
- **F2 (`newProcessCap` signature).** Every site of `processCapFor` and `newProcessCap` carries the new `commands` parameter: `processcap.go:48`, `cli.go:224`, `tierrun.go:485`, and the test stub at `tierrun_test.go:175`. The `processCap{` literals at `processcap_test.go:41` and `tierrun_test.go:177` leave `commands` nil, so they skip the filter and keep their old behaviour.

Unchecked: none.

Tool calls made:
1. Grep over docs, pattern and gatecli/gateslot for stale "after the tiers" wording. My context held only the line numbers the review named, not every site.
2. Grep for `processCapFor`, `newProcessCap(` and `processCap{` to find every caller and literal of the changed signature.
