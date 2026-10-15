# Show the orch context size in the prime overview window

`orch-context-overview` · Task · enhancement, orch, batten · running · Plan-Bouncer

## Ask (operator, 2026-10-14)

The operator wants the orch's context size easy to see, in the prime's overview window beside the batten runs.

## Today

- The prime's overview window runs `lyx batten overview` (`internal/battencli/overview.go`), one line per batten run in flight.
- The orch's context size is read by the distill watcher (`internal/orchengine/watcher.go`, `ContextTokens` in `internal/shuttleengine/claudeengine/usage.go`) and stored as `LastContextTokens` in the orch state (`internal/orchengine/state.go`).
- It is shown only by `lyx orch status` (`context_tokens`, `soft_threshold_tokens`, `threshold_tokens`); no pane or statusline shows it.

## Sketch

- The batten overview gains a line for the orch: context tokens against the soft and hard thresholds, plus the watcher's phase and cycle count, read from the orch state file it already sits beside (files only, as the overview reads today).
- Dependency direction to check: batten's overview reading orch state may cross a layer; if so, the orch line comes from a provider the orch module registers, or the runs strand runs a composed view.
