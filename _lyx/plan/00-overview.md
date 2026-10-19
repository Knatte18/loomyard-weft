---
format: 5
approved: false
language: go
---

# Plan: hub-workspaces

The operator opens a hub through one VS Code workspace file, `<hub>/<RepoName>.code-workspace`, listing the prime (named after its branch), `Bolt`, and every live task pair at its anchor directory in name order.
`fabricengine` owns one regeneration function, called by the prime spawn and by every fabric verb that creates, removes or switches a worktree, under a bounded hub-level lock.
The `_portals` container, whose only consumer was the old workspace, is retired: no verb creates, restores or removes a portal link, Reconcile reports a launcher-only restore as `launchers_restored`, and `lyx fabric reconcile` sweeps the retired links and the old `_launchers` workspace file from existing hubs.

## Card Index

1 — hub-workspace-regenerate — one fabric-owned regeneration of `<hub>/<RepoName>.code-workspace`, used by the prime spawn
2 — verbs-regenerate — Clone, Add, AddLocal, Remove, Prune apply, Checkout and Reconcile regenerate the file, a failure only logged
3 — portals-unwired — no verb creates, restores or removes a portal; Reconcile reports `launchers_restored`
4 — portal-helpers-delete — delete the portal helpers and accessor left without callers
5 — retired-artifact-sweep — Reconcile sweeps retired portal links and the old workspace file; regeneration warns while they exist
6 — workspace-docs — docs name the hub workspace file and retire `_portals`

## Shared Decisions

- The folder listing is a pure function over parsed `git worktree list` entries, so ordering, naming and skip rules are pinned at the untagged tier; the lock, settings and verb wiring are pinned at the `integration` tier through `hubforge`.
- Tests that used a file planted at a portal path to make `Add` fail after both worktrees exist plant it at the pair's launcher directory, which fails at the same step.

## PATTERN changes

- `PATTERN-fabric-write-containment` — changes: Writing under `_launchers`, or the hub workspace file, from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`.
- `PATTERN-hub-containment` — changes: Linking hub-level containers or writing the hub workspace file: from the hub only, never into a worktree.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/fabricengine/... ./internal/ideengine/... ./internal/fabriccli/... ./internal/hubforge/... ./internal/battencli/... ./internal/lyxcwd/... ./cmd/lyx/...
go vet -tags tmux ./internal/fabricengine/... ./internal/ideengine/... ./internal/fabriccli/... ./internal/hubforge/... ./internal/battencli/... ./internal/lyxcwd/... ./cmd/lyx/...
go vet -tags llm ./internal/fabricengine/... ./internal/ideengine/... ./internal/fabriccli/... ./internal/hubforge/... ./internal/battencli/... ./internal/lyxcwd/... ./cmd/lyx/...
go test ./internal/fabricengine/... ./internal/ideengine/... ./internal/fabriccli/... ./internal/hubforge/... ./internal/battencli/... ./internal/lyxcwd/... ./cmd/lyx/...
go test -tags integration ./internal/fabricengine/... ./internal/ideengine/... ./internal/fabriccli/... ./internal/hubforge/... ./internal/battencli/... ./internal/lyxcwd/... ./cmd/lyx/...
```
