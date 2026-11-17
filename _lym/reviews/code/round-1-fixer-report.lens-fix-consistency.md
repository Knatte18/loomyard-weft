```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the change set.

Findings:

- LOW. Changed text: `internal/reedengine/bindings.go:212`, "the RC button is drawn only while it is set".
  - The line is still true but now incomplete: the button is also drawn only for a named strand in a session with `@lym_button_dir`.
  - It is not contradicted by the changed docs. This is optional.

No other twin was left stale.
- The `bar.go` header, the `remoteControlShownCondition` comment, `doc.go:212`, `rcShownWhen` and `bar_test.go`'s doc comment all name `@lym_strand_name` and `@lym_button_dir` alike.
- `paneoptions.go`, `template.yaml` and the slack `--dir` help agree with `doc.go`.
- `statusFormatActions` still keys the action row on `@strand`, which is correct and untouched.

Calls: one grep over non-test `.go`, `.md` and `.yaml` files for "RC button" and "RC click" lines mentioning strand, drawn, pinned or run directory. My context held the changed files but not the sibling files and docs, such as `bindings.go`, that might still describe the old condition.
