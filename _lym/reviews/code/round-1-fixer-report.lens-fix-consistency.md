```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fix-consistency lens, meaning whether the change set agrees with its surroundings and whether any twin of a changed text was left unchanged.

The wording of the review's twins is consistent:
- The first rule in `internal/burlerengine/prompt.go:330` still says "commit each finding's fix on its own", and the new group rule right after it qualifies it. The `doc.go` bullet and the `fixScopeRules` doc comment carry the same group, chain and earlier-commit sentences.
- `contracts/stencils/burler/burler-step-fix.md:98` no longer says "commit-per-fix".
- No other "Location points at / offending content" wording remains.
- The `gatefix`, `webster` and `weaver` "commit each fix" lines belong to other seats with their own `fix: <summary>` subject. The change set does not touch them.

The "commit-per-fix" mentions in `contracts/recipes/tapestry-recipe.yaml:452`, its copy at `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:549`, and `docs/skills.md:188` are comments or docs about a Burler row running source scope. They stay true as a loose description. The review did not name them, and the rubric does not count them as stale.

Calls made:
1. `grep -rniE` for "commit-per-fix", "commit each fix/finding" and "Location points at/offending content" across the repo, to find twins of the changed text that my context lacked.
