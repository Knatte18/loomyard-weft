```yaml
lens: fix-consistency
outcome: reported
```

Task: check the five test-only fixes (F1–F5) for agreement with their surroundings, through the fix-consistency lens.

No findings.

What I checked against my context:

- **F1.** The `UnknownKeyInAPatternCluster` row expects `parts[0].pattern_clusters[0]`. `rejectUnknownKeys` builds that path as `the proposal` + `.parts` + `[0]` + `.pattern_clusters` + `[0]`. The new `TestParse_PatternClusters` uses the same `tiergate.Require(t, 0)` and `t.Parallel()` pattern as its siblings.
- **F2.** The loop covers `part-a` plus `proposal.Project.Slug` when the project is set. In `raddleFileProposal` only the project and beam shapes set a project, and the loop runs only the tapestry, tasks and project shapes. The project shape is therefore the only extra record read. The tier-0 gate passed, so the project record does hold the mapped bullet.
- **F3.** The new table row and its `fromFile` field match the existing `withdraw` handling. The `--from` JSON uses the same title as `goodDecisionInput`, so `wantHeading` still matches. The `!tc.fromFile` guard keeps the flag form from being set beside `--from`.
- **F4.** The `beforeDecisions` and unterminated rows follow the `withEntry` convention in the same test. The fenced `Withdraws:` row matches the neighbouring rows in `TestCheckAddedDecisionsKept`. The tier-0 gate passed.
- **F5.** The fix removes `PATTERN.md` after every assertion on `got` that needs the file. The tier-1 `loomcli` run passed. The later assertions use `got`, not `withoutPattern`, so they are unaffected.
- **Twin sites.** The commit messages follow the `<target>: fix <id> — …` format. All five commits touch only test files, so no production doc or spec needs a matching change.

Tool calls made beyond my context: none.
