```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each added test against its defect:

- **F2, `TestResolveGateSpec_CleanRequiresAbsoluteWorktreeRoot`:** The `clean` case at `internal/shedrecipe/entries_gate.go:144-145` calls `requireAbsRoot` on `WorktreeRoot` before anything else. An empty root and `relative/worktree` both reach that check and get an error naming `WorktreeRoot`. With the check removed, `resolveGateSpec` returns a nil error, and `assertErrContains` fails on nil. The empty subtest name becomes `#00`, which is harmless.
- **F3, the tapestry subtest in `TestDarnVerifyBudget`:** The tapestry YAML has exactly one `Code-Burler-Fix` row, at line 533, with gates `clean` then `verify` (lines 556-559). The subtest reads gates as `[]any` of `map[string]any`, the same way the existing subtests do. Deleting the `clean` entry, reordering the gates or removing the row leaves `names` as `[verify]`, `[verify clean]` or nil. Each mismatches `[]any{"clean","verify"}`, so the test fails.
- **F1, the spec sentence:** The vocabulary matches the resolver's error text at `internal/shedrecipe/entries_gate.go:232`: `clean`, `discussion`, `plan`, `parts`, `description`, `verify`. No signature changed, so no caller is affected.

Further tool calls:
1. One Bash call read `contracts/recipes/tapestry-recipe.yaml:533-566` and the `assertErrContains` definition. My context lacked the tapestry row's gates as they stand now, and it lacked what the helper does with a nil error.
