```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set for consistency with its surroundings.

No findings.

- The F1 table change is applied to every row (struct field plus all ten literals). The new `strings` import is used. The `return` after the first `t.Errorf` guards `err.Error()` against a nil error.
- The F2 doc sentence matches the code comment and the `effortFor(cfg.Agents, "", spec.Effort, spec.Model)` call in `OpenLaunch` (command.go:433). It also matches the `TestOpenLaunch` rows the review cites.
- The wantKey `efforts.haiku` matches the `effortsKey+"."+key` form the review quotes. I did not verify it against `checkEfforts`; the gate passed.

Calls:
- `git diff HEAD~2 HEAD`: confirmed the committed change set, which my context showed only as pre-commit edits.
