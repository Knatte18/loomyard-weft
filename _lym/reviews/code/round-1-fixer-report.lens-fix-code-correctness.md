```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F6 code and test changes through the fix-code-correctness lens.

**Findings**

1. **MEDIUM** — `LackingTemplateBlocks` now also returns `"<entry>.<from>: differs from <entry>.<to>"` strings, but two things still describe `lacking` as missing template units.
   - The Warn in `internal/hubreconcile/walk.go:218` says "a seed-only config file lacks units its template holds, and reconcile never adds them".
   - Its `way_forward` says "add each with \"lym config %s\"…, or leave it out on purpose".
   - For a differing pair that advice is wrong. The operator should compare the two blocks and delete one, and "adding" a unit does nothing.
   - The doc on `Result.Lacking` (`configsync.go:203`) and the comment on the `lacking` key in `configcli.go` say the same thing.
   - The F5 test only checks the substring "lacks units its template holds", so it does not catch the mismatch.
   - Evidence: the new code appends the "differs from" strings to the same slice that `result.Lacking` carries (`migrate.go`).

2. **LOW** — in `configcli_integration_test.go`, the planted `profiles.yaml` is restored only after `runReconcileCLI` returns, and the restore is not in a `t.Cleanup`.
   - If `runReconcileCLI` fails the test, the planted file stays on disk for the following subtests.
   - `runReconcileCLI` reports failures through `t.Fatalf` in the caller, so the restore line is skipped on that path.
   - Registering the restore with `t.Cleanup` right after the write would fix it.

**Traced and correct**
- **`MigrateConfig`, equal pair:** both blocks decode to equal values, the former is deleted, and one "removed" line is returned.
- **`MigrateConfig`, differing pair:** it hits `continue` with no line and no change, so `lines` stays empty and the original bytes come back.
- **Rename-only case:** it still renames as before.
- **Non-mapping root:** `"- a\n"` returns an `ErrInvalid` error.
- **`LackingTemplateBlocks` equality:** it compares `yaml.v3` `map[string]any` values with `reflect.DeepEqual`, the same decoded form `MigrateConfig` compares, so the two agree.
- **Sorting:** the result is sorted, so the expected list `light.darn, light.loom: differs…, light.pawl, light.sampler` holds.
- **Callers:** nothing else reads the strings, and `Lacking` is report-only in `configsync.go:984`, so no write is triggered.
- **Added tests:** each fails with its defect in place.
  - The differing row of `TestMigrateConfig` fails if the former block is deleted anyway.
  - The equal row fails if the former is always kept.
  - The unparseable-file row fails if no `FileError` is returned or the file is touched.
  - The dry-run `Migrated` and `Lacking` assertions fail if the dry run omits them.
  - The `lacking` key assertion fails if the key is dropped.
  - The walk Warn test fails if the Warn is removed.

Tool calls made: none; the context already held every changed file.
