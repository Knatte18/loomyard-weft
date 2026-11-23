```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced everything outside the changed lines that reads or describes the two reworded lines. Nothing is left wrong.

**Change 1: the baseline line in `renderVerdict`** now ends "the base of this branch." (`internal/driftengine/verdict.go:58`).
- **Callers of `Verdict`:** `internal/loomcli/wiring.go:1100` and `internal/loomcli/landingdeps.go:293`. Both pass the string through without parsing it. Covered.
- **Tests outside the package:** `internal/loomcli/wiring_publish_integration_test.go:136,172` and `internal/landingshed/prreview_test.go:738`. They assert only full-scope "Check: not run." text or their own fixture, never the baseline line. Not affected.
- **Package test:** `internal/driftengine/verdict_test.go:147` is updated. Covered.
- **Docs and specs:** `contracts/specs/loom-final-summary-spec.md:70-71` describes the section without quoting its wording, so it stays correct. No doc, spec or PATTERN file quotes "Baseline: captured" or "base of this branch" (my grep of contracts, docs, pattern and PATTERN.md found none).
- **Internal texts that still say "the task's branch point":**
  - `contracts/stencils/drift/drift-template-expect.md:29`
  - `internal/driftcli/cli.go:43,95`
  - `internal/driftengine/doc.go:4`
  - `internal/driftengine/records.go:49`

  Agents, operators and developers read these, not the repository's PR readers, so the board entry doesn't reach them. Not stale.

**Change 2: the off-scope lines in `scopeText`** now say "this change is not regression-checked." (`internal/driftengine/verdict.go:100,102`).
- **Readers and asserters:** only `internal/driftengine/verdict_test.go:45`, which is updated. Covered. No other code or test matches "not regression-checked" except the `records.go:18` comment.
- **The `records.go:18` comment** ("ScopeOff records that the task is not regression-checked.") documents the record constant for developers. It isn't repository-facing, so it isn't stale.
- **Docs, specs, stencils:** nothing quotes the line.

**Rubric items:**
- **Stencil wording:** the diff changes no stencil.
- **The repository's own rules:** the diff touches no PATTERN subject (no paths, git, CLI or records format), and its comments are unchanged.
- **Declared PATTERN changes:** the directive declares none.
- **Commit messages and departures:** the one commit carries `Root-Cause:` and no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed. Whether the `Root-Cause:` trailer is adequate belongs to the handler.

Findings: none.

Tool calls:
1. Grep of contracts, docs, pattern and PATTERN.md for the block's old and new wording, plus a read of `loom-final-summary-spec.md` lines 60-80. The manifest had no search of docs or specs for the block's line wording.
