```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency check of the change set for F1–F3.

**1. LOW: `internal/shuttleengine/claudeengine/config.go:28-33`, the field docs for `PromptCacheTTL` and `PromptCacheTTLRoles` (twin left unchanged)**
The fix scoped these two keys to the main conversation in `template.yaml:3-4`, `docs/overview.md:715`, `doc.go:82` and the `promptCacheTTLEnvKey` doc. The Go fields that carry the keys still say the unscoped thing:
- "PromptCacheTTL is Claude Code's prompt-cache TTL for a role without a map entry"
- "PromptCacheTTLRoles maps a strand's role segment to Claude Code's prompt-cache TTL"

Both are now stale for the same reason F2 gave: the value sets only the main conversation's TTL.

**2. LOW: `internal/shuttleengine/claudeengine/command.go`, the rewritten `subagentPromptCacheTTLEnvKey` doc (departs from the line-break convention)**
The first line ends "...of every request outside the main" and the next begins "conversation: subagents, forks, ...". This breaks mid-phrase, which the prose rule forbids ("Never break mid-phrase to meet the limit"). The preceding commit, 8c3a7047b, was made to fix wrapped comment lines of this kind. Rewrite it as shorter sentences, or break after the colon.

The other places I checked agree with the change: `template.yaml:8`, `command.go`'s launch, resume and open docs, `doc.go:88-92`, `command_test.go:23-24` and `:382`, and the summary's remaining bullets.

Tool calls:
- `git grep` for prompt-cache TTL wording outside the files already changed, because my context had not covered the whole tree without truncation.
- `git grep` on the two `config.go` files, because their field docs were not in my context.
