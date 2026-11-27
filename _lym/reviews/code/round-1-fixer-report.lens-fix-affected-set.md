```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix's change set.

Findings left unchecked: none.

no findings

The searches turned up no member the fix missed:
- **F1:** both callers of `needsPrebuiltLym` (`tierrun.go`, `copies.go`) only reach it for runs that have invocations.
- **F2–F5:** no remaining doc or comment outside dated research reports says every tagged run prebuilds, or still uses the old "test of its targets" wording. `cmd/testtiming`'s own comments describe the harness, which still always builds.
- **F6:** nothing references the old copies test name.
- **F7:** every branch of `isDirectoryTarget` now has a row.

Tool calls:
1. Repo-wide grep for prebuild statements (non-test files): my context held only a symbol grep over `internal` and `cmd`, not the docs, YAML or pattern files.
2. Grep for the old test name and old wording: my context had no search for either after the edits.
