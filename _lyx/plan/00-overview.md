---
format: 5
approved: false
language: go
---

# Plan: fabric add after remove — leftover remote weft branch

`lyx fabric add <slug>` after a plain `lyx fabric remove <slug>` fails today whenever the leftover `<slug>-weft` branch on the weft origin diverges from the re-created one, which depends on commit timestamps.
This plan adds read-only pre-flight probes of both origins to `Topology.Add`, a leased gated replacement of a weft leftover that an `archive/<slug>/*` tag covers, and a clean typed refusal for every other leftover, surfaced by the CLI as a pre-flight failure.
`Remove` is unchanged, and no remote warp branch is ever deleted.

## Card Index

1 — leased-remote-delete — `gitrepo` deletes a remote branch only while it sits at an expected SHA
2 — gate-archived-weft-delete — `destroy.go` gains the pair-weft ownership kind, the archived-on-remote dirtiness kind and a lease SHA
3 — add-remote-preflight — Add probes both origins before its first mutation and refuses an unreplaceable leftover
4 — cli-leftover-preflight-envelope — `lyx fabric add` emits a leftover refusal as a bare pre-flight error
5 — add-replace-archived-weft — Add replaces an archived weft leftover at step 12; tests and docs

## Shared Decisions

- The archive tag is the orphan proof: a remote weft tip equal to, or an ancestor of, an `archive/<slug>/*` tag's target on origin is replaceable; anything else is refused.
- Every ancestor test counts a commit missing locally as not an ancestor; only the archive probe fetches, into `FETCH_HEAD` only.
- `SkipPush` and `SkipGit` skip every probe and the replacement.

## verify:

```
go build ./...
go vet -tags integration ./internal/gitrepo/... ./internal/fabricengine/... ./internal/fabriccli/...
go test ./...
go test -tags integration ./internal/gitrepo/... ./internal/fabricengine/... ./internal/fabriccli/...
```
