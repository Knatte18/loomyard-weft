```yaml
lens: fix-proposal
outcome: reported
```

**Fork result (lens fix-proposal):** I challenged the reviewer's F1 and F2 proposals as the fixer carried them out.

**Findings**

1. **MEDIUM — F1: the `role_prefix` link has no test, and the proposal's "declared once as a constant" requirement was not met**
   - Changed text: `segmentAssistBounds` in `internal/loomcli/wiring.go`, which calls `burlerengine.ReviewChairRole(string(perch))`.
   - The fix takes each segment's prefix from `string(perch)`, and the `Perch` constants are `"discussion"`, `"plan"` and `"code"`. The recipe YAML declares the same three strings independently, in the `role_prefix` keys of `contracts/recipes/*.yaml`.
   - `rolePrefixViolations` (`internal/loomrecipe/bouncerrows_test.go:13-61`) only checks that a segment's rows agree and that segments differ. It does not check that a value equals the perch name.
   - Concrete case: someone renames a recipe's `role_prefix: code` to `coding`. Every test still passes. The chair then runs as `coding-burler-review`, and the assistant bound is silently resolved for `code-burler-review` again, which is the F1 defect.
   - The test the fixer added pins the Go side only. It sets `code-burler-review` in `claude.yaml` and never reads the recipe's `role_prefix`.
   - The fix also relies on the one-to-one match between a perch and its segment's prefix without declaring it anywhere. A test that parses the shipped recipes and asserts each review segment's prefix equals `string(loomSegmentPerches[segment])` would close this.

**Challenged and held**

- **F1, test inputs:** `3m` for the code role and `2m` for the bare role both pass the prompt-cache-lifetime check. The test only calls `wire`, which never runs that check. The room is positive: `10m` less the start allowance and `250s` is still above zero for any plausible allowance.
- **F1, numbered roles:** `claudeengine.keepWarmCapFor` takes the base entry for a numbered role such as `code-burler-review-2`, per `keepwarm_test.go:33`, so a numbered chair resolves to the same cap.
- **F1, `ledRole` and `ReviewChairRole`:** a profile with an empty `RolePrefix` gives `burler-review`, which matches the old behaviour. `fix.go` still uses `rolePrefix`, which now delegates to `ledRole`, so its output is unchanged.
- **F1, `loomSegmentSettings` callers:** only `wiring.go` calls it, per the earlier grep, so changing the signature breaks no other caller.
- **F2:** the `zeroMeans` strings match the reviewer's. The "must not be negative" fragment that `config_test.go:613` asserts is kept. Nothing asserts the old "use 0 to defer to shuttle's run_timeout_min" text for the assist row.

**Calls made**
1. Read `internal/loomrecipe/bouncerrows_test.go` lines 1-70. My context lacked any test that ties a recipe's `role_prefix` to the perch names.
