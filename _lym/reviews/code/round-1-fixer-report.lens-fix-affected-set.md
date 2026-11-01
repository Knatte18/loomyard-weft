```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the searches for F1 and F2. F3 is a single assertion, so there was no set to check.

- **F1 (`claude_deny_agent_tool` move):** no findings. No `KeyMove` or `KeyCopy` remains for `deny_agent_tool`. Every remaining mention is the retired-key handling: `shuttleengine/config.go:84` and `claudeengine/config.go:44`. The two shuttle-layer cases in `shuttleengine/config_test.go` and the claude-layer case in `claudeengine/config_test.go` still pin that a layer holding the key loads or a local override is refused. The new configsync row covers the reconcile case.
- **F2 (bundled print flag):** no findings. The old `(-p|--print)` alternative is gone from every non-`_lym` source, and the 16 occurrences in `settings-golden.json` were all rewritten (the gate is green). The `rows_integration_test.go` rows cover the new deny and allow cases.
- **F3 (duplicate Agent check):** not rerun, since it is one assertion in one test.

Nothing is left unchecked.

Calls made:
1. grep for `claude_deny_agent_tool|deny_agent_tool` across go, yaml and md files. I needed to find sites outside the files I had read.
2. grep for `(-p|--print)` and `claude -p` across go, json and md files. I needed to find the pattern's other copies and the other tests on the row.
