```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Checked:
- No twin of the recipe list was left stale. A repo-wide search for the old "darn, sampler, pawl", "sampler, pawl and batten" and "reserved as a recipe name" wording finds only the two updated glossary lines, 24 and 37.
- The motif glossary row now agrees with the `cartoon` row and with the "names reserved" line, which still describe `cartoon` as a later block beside `motif`.
- Every recipe name in the glossary list at line 24 matches a file in `contracts/recipes`. That directory holds `motif-recipe.yaml`, which I saw earlier in `ls`.
- `c.runID` is now set in `TestRecipeKeyedWiring`. That agrees with `loomCommitStatusDeps(location, c.runID)`, and the table's `wantFinished` values (tapestry empty, motif draft) match `finishedBoardStatus`.
- The new `claim_test.go` expects `claimKeep` for a draft status. That matches `settleClaim`: `IsRunStatus` is false for the word "draft", so it returns `claimKeep` before any batten state is read.
- Both the `TestSeedDiscussedRecord_Real` doc comment and the file header in `discussionrecord_integration_test.go` stay accurate with the added rows; the failure rows are covered by "asserts what the seam seeds".

Tool calls made:
1. One `grep` over `docs`, `internal` and `contracts` for stale twins of the old recipe-list and motif wording. My context held only the glossary lines the fixer had edited, not the rest of the repo.
