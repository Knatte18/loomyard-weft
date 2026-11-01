```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes (F1 rows, F3 subtest, F2 doc line) for correctness.

No findings.

- **F1, unmarked row** (`active: "cautious"\n`, batcher, migrate): the strip compares `active` against the former value `"cautious"` and strips it. The emptied file is deleted and the marker is written. The test expects exactly that, and its `resolutionChanges: true` holds because the worktree resolves `cautious` before the strip and the template's `wide` after. With the `formerTemplateValues["batcher"]["active"]` entry deleted, `cautious` differs from the template's `wide`, so it is kept. `wantStripped`, `wantDeleted` and `resolutionChanges` then all fail, so the row catches the defect. The gate run passed.
- **F1, marked row**: the strip is skipped on a marked board, so the file stays byte-identical and `wantFile` matches. The marker check is unaffected (`marked` is true). The gate run passed.
- **F3 subtest**: it decodes `batcher.ConfigTemplate()` into a struct with `batchifier`, `max_cards` and `budget`. A missing profile gives zero values and fails the budget assertion. A budget drift in either profile fails, and so does a `max_cards` or `batchifier` mismatch. `yaml` is already imported in the file, and the gate run passed.
- **F2**: documentation only, with no code path.

No tool calls made; the context already held every changed file.
