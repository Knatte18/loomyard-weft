---
format: 5
approved: true
language: go
---

# Plan: Prompt-cache TTL per agent role

Every lyx agent runs as a Claude Code main session, so on a subscription each writes its prompt cache at the 1-hour price even when it never idles past 5 minutes.
Shuttle sets the TTL per role when it starts a session, on both the launch line and the resume line, through `CLAUDE_CODE_PROMPT_CACHE_TTL`: 1 hour for the loom driver and the Webster Master, 5 minutes for every other role, configurable per role in `shuttle.yaml`.
`shuttleengine` carries the two config keys uninterpreted; `claudeengine` resolves, validates, logs and applies the value.

## Card Index

1 — shuttle-ttl-config — `shuttle.yaml` and `shuttleengine.Config` gain the TTL default and the per-role open map
2 — configreg-shuttle-open-map — the `shuttle` registry entry declares the per-role map an open map
3 — claudeengine-prompt-cache-ttl — `claudeengine.Prepare` resolves, validates and logs the role's TTL and puts it on both pane lines

## Shared Decisions

- The map is keyed by `Spec.Role`, matched exactly; an empty role or a role without an entry takes `claude_prompt_cache_ttl` (D3).
- Only `5m` and `1h` are accepted, exact and case-sensitive, for the default and for every map value, and the whole configured set is checked on every `Prepare` (D4).
- No caller's `Spec` changes, and the subagent bucket (`CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`) is not set.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./internal/shuttleengine/... ./internal/configreg/... ./internal/webstercli/...
```
