```yaml
lens: fix-way-forward
outcome: reported
```

Checked the ways forward the fix introduced (D7 after-filing refusal, D10 halt, D6 skip end, D4 guard bound).

no findings

The D7 way forward can be reached. A filed part runs `darn` (`RecordWrite` row, `contracts/recipes/darn-recipe.yaml:78`) or `tapestry`, whose `motif` block row (`contracts/recipes/tapestry-recipe.yaml:123`) supplies the record row. `lym loom decision add` judges that row from the seed recipe (`internal/loomcli/decision.go:1034-1040`). So once a released part's run passes its record row, the verb records the decision.
The D10 halt goes through the halt path Raddle-File already has.
The D6 skip can end because the mark is in `shedrun.RunDir`, and `removeRunRecords` deletes that directory (`internal/battencli/cleanup.go:391`).

Calls made:
- grep of the recipe engine rows and of decision.go's record-row test, to learn whether a filed part's run has a record row; the context lacked the part recipes.
- read of tapestry's header and block row and of decision.go:1025-1050, to confirm how the verb decides whether a run has a record row; the context lacked both.
