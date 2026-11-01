```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1:** The gate-name vocabulary appears once in docs, at `contracts/specs/shed-recipe-spec.md:74`, now updated. The only other site is the error string at `internal/shedrecipe/entries_gate.go:232`, which already lists all six names.
- **F2:** `clean` is one `case` in the resolver, and the new test covers its absolute-root refusal with an empty and a relative root.
- **F3:** `clean` appears on four recipe rows. Three are pinned: darn lines 106 and 207 and sampler line 151 by `TestDarnVerifyBudget`, and tapestry line 556 by the new subtest. I did not check that the darn line 207 row is the Code-Burler-Fix row the subtest iterates over.
- **Left unchecked:** `verify` gates without `clean` at `darn-recipe.yaml:124` and `tapestry-recipe.yaml:439`. The review's Rejected list already covers those rows (dirty-tree checks elsewhere), and they are outside these findings.

Tool calls:
1. grep for the vocabulary sentence across md/go/yaml, to rerun F1's search, because the context held only the spec.
2. grep for `name: clean` and `name: verify` in `contracts/recipes/`, to rerun F3's search, because the context held only the tapestry row.
