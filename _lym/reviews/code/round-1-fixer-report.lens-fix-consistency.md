```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the F1 and F2 change set for agreement with its surroundings (fix-consistency lens).

- The F1 row `PrefixLongerThanForm` has the same expected string as `NoRefuseList` and `GoTestRefused`. That matches `compileCheckSentence`, because a four-word prefix cannot refuse a three-word form.
- The F2 wording in `docs/overview.md` matches the `Rows` and `compileCheckSentence` doc comments in `internal/shellguard/rows.go`.
- A search for "package-scoped compile check" across `.md` and `.go` files, excluding `_lym`, found no other statement that offers the steer unconditionally. The only hits are the test comment, the `compileCheckForms` comment and the overview line.

Tool calls:
1. Grep for "package-scoped compile check" in `.md` and `.go` files outside `_lym`, to look for an unchanged twin of the overview sentence. My context held only the overview line, not the rest of the repo.
