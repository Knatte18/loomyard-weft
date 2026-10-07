---
format: 5
approved: true
language: go
---

# Plan: run-pushes — Go pushes the task branch with the run records

A task pair's branch must be on the remote after every loom transition, whatever made its commits, and a status push must recover by itself from a rejection that is not a real divergence (#418).
fabricengine gains one both-sides anchored push under the absorbing push lock, with a bounded wait for status pushes and one fetch-checked retry on a rejection; loom's status seam, `lyx loom commit-records`, `lyx fabric push` and `lyx fabric sync` use it, and batten's seam keeps pushing the records side only through `PushAnchored`, which gains the lock and the retry.

## Card Index

1 — lock-bounded-wait — `internal/lock` acquires a write lock within a bounded wait
2 — gitrepo-has-unpulled — `Repo.HasUnpulled` tells a stale-ref rejection from a real divergence
3 — fabric-pair-anchored-push — `PushPairAnchored`, and `PushAnchored` under the absorbing lock with one retry
4 — loom-pushes-both-sides — loom's status seam and `commit-records` push the task branch; split refusal row; PATTERN entry
5 — fabric-push-sync-both-sides — `lyx fabric push` and `lyx fabric sync` push both sides

## Shared Decisions

- Side names `warp` and `weft` appear only in fabricengine's error texts and fabriccli; `loomcli`, `battencli` and `statuscommit` name no side.
- Every push stays rebase-free: no card adds a rebase, merge or force on either side.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/lock/... ./internal/gitrepo/... ./internal/fabricengine/... ./internal/battencli/... ./internal/loomcli/... ./internal/fabriccli/... ./internal/statuscommit/... ./internal/pattern/... ./cmd/lyx/...
go test -tags tmux ./internal/loomcli/...
go vet -tags llm ./...
```
