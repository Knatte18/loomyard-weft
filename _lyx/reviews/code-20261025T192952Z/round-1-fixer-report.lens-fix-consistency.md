```yaml
lens: fix-consistency
outcome: reported
```

Task restated: fix-consistency check of the F1 and F2 change set.

**Findings**

1. **LOW**, `internal/battencli/overview.go`, `childRecipes` field and `resolveChildRecipe`.
   - The twin cache was left unchanged. F1 now drops `pairs` when a run leaves Run-Shed, but `childRecipes` is still keyed by slug and never dropped (`overview.go:187`, `:252`, `:263`).
   - After a pair is moved and a new run starts in it, the recipe cached from the old pair's seed is still used without re-reading. The child number can then come from the old run's recipe.
   - The "a seed is write-once" comment is true per seed file, but the cache outlives the pair's seed, which the new `pairs` comment now treats as replaceable.
   - Whether to fix this is the fixer's call. The review did not name it, and the rubric excludes unrequested improvements unless the fix made things worse. Here the fix newly makes the two caches disagree about lifetime.

2. No other inconsistency found.
   - No doc or pattern file mentions the `pairs` cache lifetime. The grep of `doc.go`, `docs/overview.md`, `PATTERN.md` and `pattern/` found nothing.
   - The `commandOf` doc comment, the `orchCLI` comment "populated by PersistentPreRunE" and the `runsWindowTitle` field comment agree with the new tier1 test's expected `beam: prime` and `beam: <slug>`.
   - The review text says the title is "beam" then a space-separated label, but the code and field comment say `beam: <label>`. The test follows the code, which is correct.

**Tool calls**
- Call 1 (`grep` over `doc.go`, `docs/overview.md`, `PATTERN.md`, `pattern/` and `internal/battencli/*.go`): my context did not show whether any doc states the `pairs` cache lifetime, nor the other uses of `childRecipes`.
