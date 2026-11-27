```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking that the F1–F3 fixes cover every member of each finding's set.

Findings left unchecked: none.

1. **LOW**: F2's set, the `PromptCacheTTL` and `PromptCacheTTLRoles` field docs at `internal/shuttleengine/claudeengine/config.go:28-33`.
   - **Summary:** both docs still describe the role TTL without limiting it to the main conversation. The fix added that limit to the `template.yaml` comments for the same two keys, the `promptCacheTTLEnvKey` doc, `doc.go` and `docs/overview.md`.
   - **Evidence:** `config.go:28` reads "PromptCacheTTL is Claude Code's prompt-cache TTL for a role without a map entry". `config.go:31` reads "PromptCacheTTLRoles maps a strand's role segment to Claude Code's prompt-cache TTL". `template.yaml:3` now reads "prompt-cache TTL of the main conversation for every role without an entry".

The F1 and F3 sets showed no missed member.

Tool calls:
1. `git grep` for subagent and TTL scope claims, because my context had only an earlier grep taken before the edits, with a narrower pattern.
2. `git grep` for sites naming the role prompt-cache TTL, because `config.go` and other non-edited sites were not in context.
3. `git grep` in test files for docs listing the TTL exports (F3), because my context lacked a sweep of all test files.
4. `sed -n` of `config.go:25-34` plus a grep of specs and shuttle config, because the field docs' full text was not in context.
