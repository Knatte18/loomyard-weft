---
format: 5
approved: true
---

# Plan: the reed viewer follows the run

A reed session gets a per-session follow state, the session option `@lyx_follow`, on by default, which replaces the viewer-position heuristics: a new agent window takes the viewer exactly when follow is on, an attach beside another client selects the relevant window while follow is on, and every `lyx loom start` entry turns follow on and selects the run's relevant window.
Reed's manual window-change bindings turn follow off, and one follow button on the session line toggles it, running the new `lyx reed follow` verb to jump back to the run.

## Card Index

1 — follow-state-focus — the `@lyx_follow` state, set on at boot, decides whether a new agent window takes the viewer
2 — attach-follow — attach selects the relevant window beside another client while follow is on
3 — follow-run-op — `Engine.FollowRun` turns follow on and selects the relevant window
4 — loom-start-follows — every `lyx loom start` entry calls `FollowRun`; `lyx loom resume` does not
5 — reed-follow-verb — `lyx reed follow`, told its geometry, turns follow on and selects the relevant window from tmux facts
6 — follow-bindings-button — manual window-change bindings write follow off; the follow button toggles it

## Shared Decisions

- Follow is off only when `@lyx_follow` reads `off`; unset reads on, and a failed read counts as off for that one decision, logged at Warn.
- A failed write of the option or a failed select is logged and never fails the operation it rides on, except in `lyx reed follow`, whose failures are its own error envelope.
- `relevantWindow` stays the rule for state-holding paths; `followTargetWindow` is the same rule read from window options for the told-geometry verb.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./cmd/lyx/...
go vet -tags tmux ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./cmd/lyx/...
go vet -tags llm ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./cmd/lyx/...
go test ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./cmd/lyx/...
go test -tags integration ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./cmd/lyx/...
```
