---
format: 5
approved: true
language: go
---

# Plan: bolt-and-store

Make lyx's own git writes to Bolt and to the hub's object stores correct and quiet.
Stencil and spec seeding moves a board copy forward only, by the build ancestry or commit time recorded in the file's banner, and only from a clean, stamped production binary; every seed commit names the binary that made it.
Seeding, `hubreconcile`'s hub-wide config write and `lyx fabric reconcile`'s heal pull Bolt first and write nothing when it cannot be brought up to date, a seed commit that conflicts at push is dropped behind the destruction gate and regenerated, and `lyx fabric reconcile` fails when its hub-config or warp-binding push failed.
Store gc at pair removal skips while another pair's reed session is live and prunes loose objects older than a day.

## Card Index

1 — buildvcs-commit-time — `Identity` carries the commit time and gains `Clean` and `Label`
2 — buildinfo-production-channel — a production channel beside dev, stamped by the deploy tool's production route
3 — stencilstore-forward-refresh — three modes, writer keys in the banner, the forward-only refresh rule and `WritesDue`
4 — stencil-source-ordering — `StencilSource` answers the ordering by ancestry or commit time and stamps the running binary
5 — gitrepo-bolt-primitives — upstream SHA, commit detail, cherry-pick and the rebase-failed sentinel
6 — seed-commit-signature — the shared binary label, seed and sync subjects and the seed-commit recognizer
7 — bolt-pull-first-seed-drop — `Bolt.PullThenCommitWritten` over an ordered list of writes and the gated seed-commit drop
8 — prerun-seed-pull-first — the root pre-run gates on a due write, pulls Bolt first and labels its commit
9 — stencil-sync-message — `lyx stencil sync` commits under its own labelled subject
10 — hubreconcile-pull-first — the hub-wide write pulls first and leaves the stamp absent when it cannot land
11 — fabric-reconcile-push-failure — a hub-config or warp-binding commit or push failure fails the verb
12 — housekeeping-inflight-prune — gc skips under another pair's live session and prunes day-old loose objects
13 — bolt-push-seed-drop-retry — Bolt's push drops seed commits and retries once on a rebase conflict, recorded in the envelope of each fabric verb that holds a record

## Shared Decisions

### Decision: pull-first-is-a-sibling-verb

- **Decision:** the pull-first commit is `Bolt.PullThenCommitWritten`, a new verb beside `Bolt.CommitWritten`, never a re-sign of it.
- **Rationale:** `configcli`'s hub-wide config edit also commits through `CommitWritten` and must keep working offline; the record's D4 names seeding and the hub-wide write only.
- **Applies to:** cards 7, 8, 10, 11

### Decision: probe-rides-on-topology

- **Decision:** the in-flight probe is set on `Topology` through `SetInFlightProbe`, and `Topology.Remove` keeps its signature.
- **Rationale:** `Remove` has thirty-two caller files and `pairteardown`'s chokepoint test pins its arity; production reaches `Remove` only through `pairteardown`, which sets the probe for both callers of removal.
- **Applies to:** card 12

### Decision: sync-keeps-its-commit-path

- **Decision:** `lyx stencil sync` keeps committing through `CommitSeededStencils` and does not pull first; only its subject changes.
- **Rationale:** D3 makes a sync commit one D5 never drops, and D5 accepts that a conflict on it fails the push as today for the operator to resolve.
- **Applies to:** card 9

### Decision: skip-warns-once-in-bolt

- **Decision:** the fetch-failure, dirty-board and divergence Warns are logged inside `Bolt.pullFirst`, once, and callers only read the returned skip.
- **Rationale:** three callers would otherwise repeat the same way-forward text; D6 is then checked at one site.
- **Applies to:** cards 7, 8, 10, 11

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
