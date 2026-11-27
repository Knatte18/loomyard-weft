```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the F1 and F3 changes (F2 is prose only).

no findings

- **F1, dirty-worktree row** (`scopes_integration_test.go`):
  - The added `commitChange` writes content that differs from the seed `package other\n`, so the commit is not empty.
  - `--all` then records tier0 whole-green at a commit other than `f.mergeBase`, and a failed `--all` still stops the row through `t.Fatalf`.
  - If the gate wrongly used that green ancestor on the dirty tree, the derivation line would name the new commit and `wantDerivation(..., f.mergeBase, "tier0")` would fail.
  - The sibling row runs the same commit and `--all` sequence on a clean tree and gets `green` as its base, so dirtiness is what separates the two outcomes. I reasoned this through and did not run it with the defect in place.
  - No signature or caller changed.
- **F3, `scenarioHub` comment:** comment only, with no behaviour change.

Further tool calls: none.
