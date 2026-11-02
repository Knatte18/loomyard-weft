```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (shorten the `default_recipe` comment in `template.yaml`, dropping the recipe-name list).

no findings

- The new comment text matches the `Config.DefaultRecipe` doc at `config.go:32` and `config.go:85`, so the template and the code agree.
- No other copy of the dropped list "darn, tapestry or sampler" remains outside `.lym` and `_lym`.
- `docs/overview.md:364` still says "`darn` in the template". That stays true, because the template still sets `default_recipe: darn`.
- The gate for `./internal/boardengine` passed with no failures, so no test pins the old comment text.

Tool calls:
1. `grep -rn` for the old and new comment text across the repo. My context did not hold any other copy of the template comment.
