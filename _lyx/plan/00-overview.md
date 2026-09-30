---
format: 5
approved: false
language: go
---

# Plan: fabric remove pushes the archive tag after its dirty checks

`Topology.Remove` pushes its `archive/<slug>/<tip>` tag and tears down the portal and launchers before its no-force dirtiness checks run.
A teardown refused for a dirty sibling therefore leaves a tag that misses the uncommitted records, and the resume after committing them pushes a second tag.
This plan moves every refusal, dirtiness checks and status probes included, ahead of the archive and the archive ahead of every removal, deletes the reconcile-remedy machinery the reorder makes dead, and rewrites the tests and comments that pinned the old order.
It is one card because the reorder breaks every old-order test at once, and the replacement tests share files with them.

## Card Index

1 — refuse-before-archive — reorder Remove so refusals precede the archive, delete the stranded-teardown remedy, rewrite the old-order tests and docs

## Shared Decisions

- `force` answers dirtiness only: it skips the two dirtiness checks and never the archive.
- Refusal base texts and the `ErrPairSiblingDirty`/`ErrArchiveFailed` sentinels stay unchanged; only the reconcile suffix goes.
- Archive tags already pushed by the old order are left alone.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/fabricengine/... ./internal/battencli/... ./internal/fabriccli/...
! grep -rn nameStrandedPortalTeardown --include='*.go' .
```
