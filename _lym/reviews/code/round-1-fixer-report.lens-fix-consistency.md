```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the round-1 change set.

**Findings**

1. LOW. `internal/shuttleengine/claudeengine/doc.go:280` says the row denies "`-p` or `--print` among its arguments". The pattern now also denies any single-dash word holding `p`, such as `-pc`, `-cp` and `-dp`, so the doc undersells it.
   - The rewritten comment at `internal/shellguard/rows.go:143` was updated.
   - The doc.go twin was not.
   - The same row's `Forms` at `rows.go:346` (`{"claude","-p"}`, `{"claude","--print"}`) are untouched. They are probably only display or steer forms, so I did not verify them.

2. LOW. The false-denial class widened and no comment says so.
   - The old pattern falsely denied a subcommand's own `-p`.
   - The new pattern also denies any single-dash flag word with a `p`, such as `claude -dp x` or `claude -pr`.
   - The doc.go:282 guardrail sentence names only the `-p` case. The `rows.go` comment does not list the wider false denial either.

No inconsistency found in the F1 twins. The one remaining `claude_deny_agent_tool` reference outside the new test row is `shuttleengine/config.go:84` and `config_test.go:198,240`. They are consistent with the key being retired. The golden file was updated for all 16 occurrences, and the gate is green.

**Calls made**
- Bash grep for `claude_deny_agent_tool` and the old pattern text: I needed to find unedited twins of the changed text.
- Bash grep for print-flag mentions in Go and markdown: same reason, with a wider spelling.
- Read `claudeengine/doc.go:279-283`: the grep showed that line only truncated.
