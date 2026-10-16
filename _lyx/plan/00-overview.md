---
format: 5
approved: false
language: go
---

# Plan: lyx shuttle open chooses the permission mode

`lyx shuttle open` gains `--permission-mode` (`bypass`, `prompt`, empty for the engine's interactive default), so an operator session opens in bypass under its lyx strand name instead of being relaunched through `lyx reed add`.
The mode travels blind through `OpenSpec.PermissionMode` and `Runner.Open`, and `claudeengine` alone validates it with the existing interactive rule and realizes `bypass` as `--dangerously-skip-permissions` on the fresh and the resume line.

## Card Index

1 — open-spec-permission-mode — `OpenSpec` gains `PermissionMode`, carried blind to `OpenLaunch` and recorded on the opened-session log line
2 — claude-open-permission — `Claude.OpenLaunch` validates the mode and `buildOpenCmd` adds the skip flag to the fresh and resume lines
3 — open-permission-flag — `lyx shuttle open --permission-mode` fills `OpenSpec.PermissionMode`, with help text and the overview row

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shuttleengine/... ./internal/shuttlecli/... ./cmd/lyx/...
go vet -tags tmux ./internal/shuttleengine/... ./internal/shuttlecli/... ./cmd/lyx/...
go vet -tags llm ./internal/shuttleengine/... ./internal/shuttlecli/... ./cmd/lyx/...
go test ./internal/shuttleengine/... ./internal/shuttlecli/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/shuttlecli/... ./cmd/lyx/...
```
