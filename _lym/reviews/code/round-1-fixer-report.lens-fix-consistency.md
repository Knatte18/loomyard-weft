```yaml
lens: fix-consistency
outcome: reported
```

Findings (lens fix-consistency):

1. **LOW**: `internal/testcover/coveragemap.go:55` and `internal/testcover/testselect.go:75`.
   - Changed text: `mapMiss` now misses on `BuildFailure` only when `len(m.Tests) == 0`.
   - Evidence: the `BuildFailure` field doc says only "the error of a coverage build that failed". Nothing there says a record may sit beside test entries.
   - `writeFailed` at `internal/gatecli/testmappackage.go:372` still clears `Blocks`, `Funcs` and `Tests`, so the writer never produces that combination. The reader's rule is ahead of the writer.
   - A reader of the field doc would not expect the new case.
   - `MissBuildFailure`'s text "coverage map records a build failure" now fits only the no-test-entries case, and the `mapMiss` doc comment says so.

2. **LOW**: `internal/gatecli/redundancy.go:305`.
   - Changed text: the selector's `mapMiss` rule.
   - Evidence: this check still treats any `m.BuildFailure != ""` as a failed map, whether or not the map holds tests. It is the redundancy report's own twin of the selector's rule.
   - The two now disagree about a map with a failure record and test entries.
   - Whether the disagreement matters depends on how D5's upkeep plan writes that field, which I cannot see. The twin was left unchanged.

3. No finding for the import rule. The `MissTestFile` detail text and the `compareTestSources` doc agree with the new behaviour. No doc outside the changed files states the old "adds or removes an import" miss for test files.

Calls made:
- grep over `*.go` and `*.md` for the old import and build-failure wording, to find stale twins in docs and other packages.
- grep over non-test Go files for `BuildFailure`, to find other consumers of the field.
