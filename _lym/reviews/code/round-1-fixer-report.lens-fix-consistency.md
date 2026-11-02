```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 template-comment change against its siblings for disagreement or an unchanged twin.

No findings.

- The only edit is `internal/boardengine/template.yaml:3`, which now reads "recipe a task with no recipe field runs".
- That text matches the `Config.DefaultRecipe` doc comments at `internal/boardengine/config.go:32` and `internal/boardengine/config.go:85`.
- No other copy of the old wording "darn, tapestry or sampler" remains in the tree. The `docs/overview.md:364` sentence is consistent with it and was rejected by the review as a different reader's statement.
- The earlier `lym gate test ./internal/boardengine` run passed with the changed template, so no golden or embed test pins the old comment.

Tool calls made (1):
1. `grep -rnE` for the old and new comment wording across the tree. My context did not show the other places the wording might appear.
