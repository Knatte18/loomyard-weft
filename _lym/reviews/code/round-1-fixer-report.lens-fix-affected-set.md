```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings checked, with the search for each:
- **F12** (stale refresh wording): a repo-wide grep for "stale packages", "builds a map for" and "stale package" now matches nothing outside `docs/research`. A grep for `test_map_batch` and `TestMapBatch` shows `doc.go`, `testmap.go`, `config.go` and `template.yaml` all carrying the new wording. `docs/overview.md:827` only restates the `--max <test_map_batch>` command, which the change leaves true.
- **F8 and F9** (callers of `PlanUpkeep`): the only production caller is `internal/gatecli/testmapplan.go:78`. It passes `Declared` from `mappedTests`, so the new undeclared-entry drop and the failure clearing cover every site.
- **F11** (`BuiltAt` fallback): no `BuiltAt`/`MeasuredAt` fallback and no "counts as measured when its map was built" text remains in the code or docs.
- **F10** (`r.measures` append): the append is now at a single site, after the dirty and changed-under-tests skips.

Findings left unchecked: F4, F5, F6, F7, F13 and F14. They add or extend tests and have no set premise to rerun.

Tool calls:
1. Grep for the stale refresh wording. The review named only three sites, and the fix had to cover any other.
2. Grep for `test_map_batch` and `TestMapBatch`. This lists every site that mentions the key, for F12.
3. Grep for `BuiltAt` in the testmap and testcover non-test sources. This lists every consumer of the measurement-time fallback, for F11.
4. Grep for the `PlanUpkeep(` callers and for text about the `MeasuredAt` fallback. This confirms the callers of F8 and F9 and shows no leftover fallback text.
