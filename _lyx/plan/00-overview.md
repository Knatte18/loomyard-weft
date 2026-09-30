---
format: 5
approved: true
language: go
---

# Plan: Publish — verify the tree after merging the parent

The landing producers in `internal/landingshed` merge the parent branch into the task branch through `internal/mergeresolve` and then push (`Publish`) or land on the parent and push it (`Finalize`) without compiling or testing the merged tree.
A merge can compile cleanly and still break tests.
This plan adds a post-merge verify gate to both producers: after every parent merge-in that changed the task tree, or while a pending-verify marker from an earlier unverified merge exists, the plan's `## verify:` command runs in the task worktree, and a failure halts the run as Stuck before anything leaves the worktree.
The shell runner is extracted from webster into one shared leaf package, loomengine gains scratch accessors for the marker and the output file, and `internal/loomcli` tells landingshed the command (read from the plan at call time) and both paths.

## Card Index

1 — verifyrun-runner — create the shared in-process shell-command runner package
2 — webster-shared-runner — route webster's bisect verify through the shared runner
3 — loomengine-verify-paths — add scratch accessors for the verify marker and output file
4 — landingshed-verify-gate — add the told Deps fields and the post-merge verify gate
5 — publish-verify-gate — run the gate between Publish's merge-in and its push
6 — finalize-verify-gate — run the gate after every Finalize merge-in, before the parent-side merge
7 — loomcli-verify-wiring — tell landing the plan's verify command and the two scratch paths

## Shared Decisions

- The shared runner lives in a new leaf package `internal/verifyrun` (stdlib plus `internal/logger`), because `internal/shell` must stay stdlib-only and landingshed must not import `internal/websterengine`.
- The gate's trigger is `!mergeresolve.Result.AlreadyUpToDate` or a present marker; a resolver `OutcomeStuck` returns before the gate, as today.
- Failure (non-zero exit or a spawn failure) is Stuck through each producer's existing `stuckOrCancelled`; a closure error, a marker write failure, an output-file create failure and a cancellation are returned errors; the merge commit is never undone.
- A nil `Deps.VerifyCommand` means no verify wiring (nil-is-absent, silent); an empty returned command skips the gate with a `logger.Warn`.
- The command is read through a closure each time the gate runs, never captured at `landingDeps` time.
- The marker and output file are told paths under loom's scratch directory; landingshed derives neither.
- No recipe row, landing.yaml key, conflict stencil or approval-path change.

## verify:

```
go build ./...
go vet -tags integration ./internal/verifyrun/... ./internal/websterengine/... ./internal/landingshed/... ./internal/loomengine/... ./internal/loomcli/... ./cmd/lyx/...
go vet -tags smoke ./internal/loomcli/... ./internal/landingshed/... ./internal/websterengine/...
go test ./...
go test -tags integration ./internal/verifyrun/... ./internal/websterengine/... ./internal/landingshed/... ./internal/loomengine/... ./internal/loomcli/... ./cmd/lyx/...
```
