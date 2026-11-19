```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. Each finding is a change a card makes that something outside its listed members still asserts or describes.

1. **BLOCKING — card 13 (hardener-recipe), the interrupt policies.** The three new rows have no `InterruptPolicies` entries, so the meta test that checks every recipe row fails on this card.
   - The table is `internal/loomshed/interruptpolicy.go`. It pins a policy for each row name, e.g. `NameCodeBouncer` at :37 and `NameCodeBurlerFix` at :52.
   - The test is `internal/loomrecipe/interruptpolicy_meta_test.go:18-29`, `TestInterruptPolicies_MatchAssembledRows`. It walks a closed recipe list (tapestry, darn, sampler, motif, raddle, arras, pawl) and requires every row New assembles to have an entry, and every entry to name a real row.
   - Card 13 lists neither `internal/loomshed#InterruptPolicies` nor that test. Either the hardener rows go unchecked, or, once added, Harden-Bouncer, Harden-Burler-Review and Harden-Burler-Fix are rows with no policy.
   - The same test's assumptions rest on `wantProducerTable` (`internal/loomrecipe/shape_test.go:85`), the pinned row-to-engine table. Card 13 does not list `internal/loomrecipe#wantProducerTable` either.

2. **BLOCKING — cards 11 and 13, card order against `TestCoverageGuard`.** Card 11 registers `"HardenFix"`, but no recipe reaches it until card 13 embeds the hardener recipe.
   - `TestCoverageGuard` (`internal/shedrecipe/coverage_guard_test.go:39-70`) fails on any registered engine that is neither reached through `loomrecipe.RecipeEngines()` nor allowlisted. So card 11's gate fails.
   - The fix needs `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines` to gain `HardenFix` on card 11. Card 13 must then drop it, because the guard also fails an allowlisted engine that is reached (the "allowlist drift" subtest).
   - Neither card lists the var. Card 11's line "TestCoverageGuard covers the new engine name" cannot hold as written.

3. **BLOCKING — cards 5, 11 and 15, the new nil seams against `envkit.NilSeams`.** `Env.PreflightHarden`, `Env.Harden` and `Env.Landing.Reap` are nil on every non-hardener wiring.
   - `envkit.NilSeams` flags every nil seam not in `nilLegal` (`internal/testkit/envkit/envkit.go:55-140`). `PreflightBatcher`, `PreflightStencils`, `PreflightDrift` and `Segments` are listed there.
   - It is called by:
     - `internal/loomcli/wiring_seams_test.go:194`
     - `internal/battencli/wire_seams_test.go:276`
     - `internal/loomcli/wiring_beam_test.go:74`
     - `internal/testkit/envkit/envkit_test.go:62`, which needs `NilSeams(FullEnv(t))` to be empty, so `FullEnv` must fill the new fields.
   - No card lists `internal/testkit/envkit#nilLegal` or `internal/testkit/envkit#FullEnv`. Card 14 says the seam tests "list `Env.Harden` and `Env.PreflightHarden` among the fields their recipes leave nil", but that list lives in envkit, not in those tests.
   - Card 5 adds `PreflightHarden`, card 11 adds `Harden` and card 15 adds `Landing.Reap`, so each of those cards fails the seam tests on its own gate.

4. **MEDIUM — card 2 (gate-harden-keys), the rendered key list.** Adding three keys to `goReadGates` changes `TiersValue`'s rendered output, which `internal/gateslot/config_test.go` asserts (it is the only test file that asserts `TiersValue()`).
   - Card 2 lists only `templateDefaults`. The test function that asserts the rendering is not listed, so its expected text goes stale.

5. **LOW — card 13, a recipe gate in `armAt`.** `armAt` calls `refuseUnservedRecipe(c.recipe)` on the `lym loom` path (`internal/loomcli/arm.go:257-263`). If that guard holds a closed list, a hardener run's `lym loom` verbs (status, `circling accept`) are refused.
   - Card 13 does not list `internal/loomcli#refuseUnservedRecipe`. I did not read its body; this is the remaining closed-list candidate on the `lym loom` path.

6. **LOW — card 20 (crucible-docs), leftover crucible references.**
   - `tools/codestats/main.go:8` and `tools/codestats/scan.go:270` describe `-exclude crucible` for a directory card 20 deletes.
   - `internal/fabricengine/mergecrucible_integration_test.go` keeps its crucible names.
   - The acceptance criterion says "no doc names crucible". Card 20 keeps only the history comments in planparser, planglyph and shuttleengine, and does not mention these.

Tool calls beyond what my context held:
- Grep of the consumers of `RecipeEngines` and `LoomBuildEngines`, the agentconfig bouncer roles, the callers of `darnTreeScope` and `darnVerifySource`, and envkit's Env field names: to find who reads each changed set.
- Read of envkit's `nilLegal` map, the coverage guard and the tree-scope test function names: the context held only line numbers.
- Two `lym quarry resolve` runs over every Edit and Uses glyph the cards name. All resolved. The first run's filter printed nothing, so I re-ran it.
- Grep of the interrupt-policy meta test and the `NilSeams` callers: to check whether the new rows and seams hit those pin tables.
- Grep of the `TiersValue` asserts, the pairteardown enforcement test, the shape table and the remaining recipe lists: to check card 2's and card 13's forced edits.
