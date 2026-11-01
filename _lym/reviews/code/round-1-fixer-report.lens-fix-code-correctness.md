```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Each changed branch, run on one concrete input:
- **Bundled print flag:** `-[a-z]*p[a-z]*` with `claude -pc hi` matches `-pc` followed by a space. With `claude -cp hi` the pattern matches `-cp`.
- **Long flag:** `claude --print=json hi` still goes through the `--print` alternative, since `-[a-z]*` cannot consume the second dash.
- **Interactive and non-print launches:** `claude -c` has no `p`, so it does not match. `claude --model sonnet` does not match, because every `--` word is excluded from the short-flag alternative.
- **Match start:** the pattern is anchored after `[[:space:]]+`, so it cannot start inside a word. `grep claude -p f` and `claudette -p` stay allowed.
- **Golden file:** all 16 occurrences in `settings-golden.json` were replaced, and the gate went green after the replacement.
- **Deny rows against the old pattern:** with the old `(-p|--print)`, the `-cp` and `-pc` deny rows would fail, so they pin the fix.
- **Allow row:** the `claude -c` allow row passes under both the old and the new pattern. It only guards against an over-broad regex, not against the original defect.

F1 (key move):
- **Caller search:** the only remaining `claude_deny_agent_tool` references are the retired-key entry in `internal/shuttleengine/config.go:84`, its two tests in `internal/shuttleengine/config_test.go`, and the new `configsync` row.
- **Run settings:** `moveRunSettings` now leaves the key in the shuttle object, and the shuttle spec retires it, so it still loads.
- **New row against the defect:** with the move row in place, `claude.yaml` would gain `deny_agent_tool`, and the `moved` line would no longer equal nil. Both the `excludes` check and the `moved` check would fail, so the new row pins the fix.

F3 (duplicate assertion): removing the `hasMatcher` helper and its only use leaves nothing dangling, since the gate compiled. `TestBuildSettings_NoAgentHookAndForkGuard` and the golden test keep the Agent-absence check.

Further tool calls:
1. `grep -rn claude_deny_agent_tool` over `.go`, `.md` and `.yaml` outside `_lym`, `.lym` and `docs/research`: to find leftover callers of the removed key move, which my context did not show.
