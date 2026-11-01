```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals behind F1, F2 and F3, all fixed as proposed.

**F1 (drop the `claude_deny_agent_tool` key move): no findings.**
- `internal/shuttleengine/config.go:84` still lists `claude_deny_agent_tool` as a retired key.
- `internal/shuttleengine/config_test.go:240` loads a layer holding it, so dropping the move breaks no load.
- The new `configsync_test.go` row fails if the move returns, because the move would write `deny_agent_tool` into the existing `claude.yaml` that the row excludes it from.

**F2 (`-[a-z]*p[a-z]*` in the headless-claude pattern): LOW.**
- **Changed text:** `rows.go`, `headlessClaudeCommandPattern`, `(-[a-z]*p[a-z]*|--print)`.
- **Summary:** the pattern now refuses any single-dash lowercase word containing `p` after `claude`, not only bundled print flags. Claude's own flags are `-p`, `-c`, `-r`, `-d`, `-v`, so none of them hit this by accident. A word that is not a claude flag does. `claude mcp add x -- npx -pkg` is refused, because the argument-word loop passes `--` and `npx` and the next word matches. So is a launched tool's flag such as `-pr`.
- **Why LOW:** the old `-p` pattern had the same reach for the exact word `-p`. The gap is a guardrail over-match on `claude mcp add ... -- <cmd> -p…`, not a hole in the deny. The row comment names no false-positive class. If the over-match matters, tighten to a known-short-flag class such as `-[cdrv]*p[cdrv]*`, or list it in the row comment as accepted.
- The golden file replacement of `(-p|--print)` was a `replace_all` over all 16 occurrences and the gate passed, so the golden is consistent.

**F3 (delete the duplicate `hasMatcher("Agent")` check): no findings.**
- The review says `TestBuildSettings_NoAgentHookAndForkGuard` and the golden test pin Agent absence. The golden test passes for every input combination, so the removal loses no coverage.

Further tool calls:
1. `grep` for `claude_deny_agent_tool` and `"deny_agent_tool"` in the Go, YAML and markdown files outside `_lym`, `.lym` and `docs/research`. My context did not show whether the shuttle spec still retires the key, or whether any other row or test still used the move.
