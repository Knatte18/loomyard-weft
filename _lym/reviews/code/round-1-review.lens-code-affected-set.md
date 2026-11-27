```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced what the moved helpers, the new fixture types and the parallel rows force outside the touched lines. Nothing outside the diff is left wrong, and the only finding is one NIT on the `scenarioHub` type comment.

**Per change, the set I found:**
- **Package-level `withSharedBuild`, `buildRecords` and `closingOf` in `test_integration_test.go`.** Defined at `test_integration_test.go:848`, `:860` and `:867`. The callers are the scenario rows and, through `buildRecords`, the scopes dotnet shared-build row. The scopes file's old `builds` closure is removed and its own build helper is renamed `scopesSharedBuild` (`scopes_integration_test.go:281`), so the names don't collide. Covered.
- **Package-level `noCarryTier1`, `wantInertEmpty`, `scopesSharedBuild` and `vetSummary` in `scopes_integration_test.go`.** At `:265`, `:272`, `:281` and `:289`. Only the scopes rows call them, and no other file in the package uses these names. Covered.
- **`scopesHub`/`newScopesHub` and `scenarioHub`/`newScenarioHub` replacing the closures.** Every call site is in the two changed files, and no other file referenced the removed closures. Covered.
- **The changed file header of `scopes_integration_test.go`** ("It is Tier 2" dropped). The test calls `tiergate.Require(t, 1)`, so the old header was the wrong one. Covered.
- **The two tests' names and timing as other documents describe them.** `docs/research/speed/test-time-measure-2026-11-27/report.md:87,167` and `docs/research/speed/tier1-test-revision-2026-11-06/report.md:226` quote figures for these tests. Both are measurement reports, which the rubric exempts, so nothing is stale. Covered.
- **PATTERN-test-speed's "Tests run in parallel" and "Test economy" rules.** Every subtest in both changed tests now calls `t.Parallel()`. My search found subtests without it only at `test_integration_test.go:2771`, `:2788`, `:2803` and `:2825`, which sit in another test the diff does not touch. Covered.
- **PATTERN-test-isolation (shared support in one `internal/testkit/&lt;kit&gt;/`).** The new helpers serve only this package, from its own `_test.go` files. Covered.
- **Declared PATTERN changes.** The directive declares none, so there is nothing to check.
- **Commit messages and trailers.** None of the three commits carries a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. What the messages say matches the diff: own hub per row, priming `--all` runs, restores removed. Covered.
- **Leftover wording about a shared hub or row order.** A grep for "shared hub", "in order", "row before", "rows below" and "later row" in both files finds only the helper comments at `scopes_integration_test.go:1297` and `test_integration_test.go:472,494`. All three mean the order of invocations, not of rows. Covered, apart from the finding below.

**Findings:**
1. NIT, consistency, `internal/gatecli/test_integration_test.go:716` (the `scenarioHub` type comment). The comment says "Every row builds its own, so rows share no state and run in parallel". Some rows build none: the standalone-worktree rows and the rows outside every hub, such as "status outside every hub answers in_hub false" and "a directory in no git worktree…". So the comment states a rule that the test's own rows break. Evidence: diff piece 003, lines 207–208 (the comment), and the rows at piece 004 line 101 and piece 005 lines 10, 20, 278 and 286, which call no `newScenarioHub`.

Tool calls:
1. A grep of the repository for docs and tests naming the two tests, plus the definition sites of the new package-level helpers and types. My context held the diff, not who else references these names.
2. An awk pass listing subtests with no `t.Parallel()`, plus a grep for leftover shared-hub or row-order wording in both files. My context held only the diff hunks, not the whole HEAD files.
