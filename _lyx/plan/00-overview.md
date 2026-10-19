---
format: 5
approved: false
language: go
---

# Plan: hub-workspaces

The operator opens a hub through one VS Code workspace file, `<hub>/<RepoName>.code-workspace`.
It lists the prime first, `Bolt` second and every live task pair at its anchor dir after that, and fabric regenerates it whenever a verb creates, removes or switches a warp worktree.
The `_portals` container, whose only consumer was the old workspace, is retired.
`lyx fabric reconcile` sweeps the retired portal links and the old workspace file from an existing hub.

## Card Index

1 — hub-workspace-file — one fabric function writes `<hub>/<RepoName>.code-workspace` under a hub lock, and the prime spawn uses it
2 — verbs-regenerate — Clone, Add, AddLocal, Remove, Prune with apply, Checkout and Reconcile regenerate the file, warning on failure
3 — reconcile-launchers-restored — Reconcile restores launchers only, skips local pairs and reports `launchers_restored`
4 — retire-portals — no verb creates or removes a portal link; the portal helpers, accessor and their tests go
5 — retired-artifact-sweep — Reconcile sweeps retired portal links and the old workspace file, reports `hub_details`, and regenerations warn while they remain

## Shared Decisions

### Decision: regeneration-is-best-effort-in-verbs

- **Decision:** Inside a fabric verb, `regenerateAfterVerb` logs a regeneration failure at Warn naming `lyx fabric reconcile` and never fails, rolls back or reports it in the verb's result; only `lyx ide spawn <prime>` returns the failure as an error.
- **Rationale:** D3 and D4: a stale folder list is rewritten by the next regenerating verb or prime spawn.
- **Applies to:** cards 1, 2, 5

### Decision: lock-wait-injectable

- **Decision:** The 5-second hub workspace lock wait is a parameter of the unexported `regenerateHubWorkspace`, and tests shorten it through `RegenerateHubWorkspaceWithinForTest`.
- **Rationale:** `PATTERN-test-speed`: a production wait is injectable so a test never waits it out.
- **Applies to:** cards 1, 2

## PATTERN changes

- `PATTERN-fabric-write-containment` — changes: Writing under `_launchers`, or the hub workspace file, from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`.
- `PATTERN-hub-containment` — changes: Linking hub-level containers or writing the hub workspace file: from the hub only, never into a worktree.

## verify:

```
go build ./...
go test ./cmd/lyx ./internal/fabricengine ./internal/fabriccli ./internal/ideengine ./internal/hubforge ./internal/battencli ./internal/fslink
go test -tags integration ./cmd/lyx ./internal/fabricengine ./internal/fabriccli ./internal/ideengine ./internal/hubforge ./internal/battencli ./internal/fslink
```
