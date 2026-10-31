```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness (F1 map entry and its callers, F2 and F7 tests).

No findings.

- **F1 callers:** `renameRetiredRows` passes the whole map to `shedengine.RenameProducers`. That function rewrites `current_producer`, `entered_row`, `pause_before`, `pause_after` and every history entry for every map key. So the second entry works with no other change. The `peers.go` `recipeRow` lookup is a plain map index and also handles it. The `arm.go` branch gives the tapestry map to any recipe that is neither darn nor pawl. No other recipe has a row named `Loom-Preflight`, so the extra entry cannot misfire.
- **F1 test:** The new `TestRenameRetiredRows` row seeds all five fields with `Loom-Preflight` and expects `Tapestry-Preflight`. Without the map entry, `namesRetiredRow` is false, nothing is rewritten, and every field assertion fails. So the row catches the defect.
- **F2 tests:** The `loom` row in `TestValidateRecipe` and the `loom` subtest in `TestWriteSeed_UnknownRecipeRefusesWithTheAvailableNames` would fail if `loom` were aliased or accepted. Both also assert the "unknown recipe" text and that `tapestry` is listed.
- **F7 edits:** The `name == "loom"` branch in `decisionswitches_test.go` now matches the renamed map key, so the tapestry case still asserts the Discussion rows instead of an empty list. The `configcli` `--set` JSON now carries the recipe key `tapestry` over the module key `loom`, and its expected error matches. In `runprofile_test.go`, `no tapestry block` matches the `holds no %s block` format in `runprofile.go:90`. Config-module `loom` keys were left alone in the cases I checked. The fixer's last gate runs passed for `configcli` and `loomrecipe`.

Tool calls made:
1. Read `renameRetiredRows` and the `arm.go` call site: my context showed only the test, not the caller.
2. Read `shedengine.RenameProducers`: my context lacked how it handles a multi-entry map.
