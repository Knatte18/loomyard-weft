```yaml
lens: fix-affected-set
outcome: reported
```

**Findings**

- LOW. Changed text: F4's rewording of `contracts/stencils/burler/burler-step-fix.md:98`. The same stale "commit-per-fix" label survives in `contracts/recipes/tapestry-recipe.yaml:452`: "(commit-per-fix, never a push)". It describes the source-scope rule, which now also commits a group of findings as one joint commit. The comment is about the Fabric Git Invariant's permitted agent commit, so it may count as a generic label rather than the rules text. A reader would still notice that it disagrees with the new rules.
  - Evidence: `grep -rniE "commit-per-fix|commit each (finding|fix)|commit each fix"` returned only that one burler-scope hit outside the fixed sites. The other grep hits are unrelated (weaver, gatefix and darn commit instructions).

**Checked, no gap**

- F5: grepping for "offending content" and "location points/names" found only the reworded `internal/burlerengine/verdict.go:66`. No other site states the old location meaning.
- F1/F2: grepping for the old commit-rule phrasings found no remaining copy of the old text. The three sites that carry the rule (the `fixScopeRules` string, its doc comment and the `doc.go` FixScopeSource bullet) are all updated. The `burler-step-fix.md` fix-scope text is rendered from the `fixScopeRules` string, so it needs no separate edit.
- F3: the only `commit each fix` assertion in `prompt_test.go` was the one removed.
- F6: not checked as a set. It has a single site, the location rule at `burler-review-format.md:59`.

**Tool calls**

1. `grep` for commit-per-fix and related phrasings across `.go`, `.md` and `.yaml`. I had no repo-wide search result for the F1, F2 and F4 sets in context.
2. `grep` for "offending content" and "location points/names". I had no repo-wide search result for the F5 set in context.
3. `Read` of `contracts/recipes/tapestry-recipe.yaml:448-455`, to judge whether the one extra hit counts as a stale sibling.
