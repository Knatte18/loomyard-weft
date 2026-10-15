---
format: 5
approved: false
language: go
---

# Plan: shed-tackler — the Go driver and its tackler as a shed layer

The Go driver and its tackler move out of `internal/loomcli` and `internal/loomengine` into a new told-geometry package, `internal/sheddriver`, which every shed-driven recipe arms by filling seams.
Loom and darn keep `lyx loom drive` as their verb, and batten's `lyx batten run` drives its own run through the same body in-process.
A new `shed` config module holds the driver and tackler defaults, and each module's own keys override it, with a one-time migration of a hub's `loom.yaml`.
The tackler stencil becomes a recipe-neutral shed stencil, and `PATTERN-shed-driver` pins the package.
Every default `driver_kind` stays `llm`; no run is switched to `go`.

The record's one carry-over entry, `migration-comments-stale-on-pinned-key-hubs`, is answered by D5's comment-refresh step, which card 9 builds; it is not separate scope.

## Card Index

1 — interrupt-policy-words — move the two interrupt-policy words from loomshed into shedverbs
2 — sheddriver-records — open internal/sheddriver with the stop key, settle records and tackler records
3 — settle-records-per-run — move the settle records to a per-run shedrun path and point every reader at it
4 — tackler-call-to-sheddriver — move the tackler call, its shuttle seam and its stencil to the shed layer, recipe-neutral
5 — drive-verb-split — keep loom's drive verb, envelope path and refusal in drivewiring.go
6 — driver-loop-to-sheddriver — move the settle loop, rule table and hand-back into sheddriver behind optional seams
7 — shed-config-module — add the shed config module with its defaults and pure resolvers
8 — module-overrides — make loom, darn and batten keys overrides of shed.yaml and resolve per recipe
9 — loom-override-migration — migrate a hub's loom.yaml defaults and comments once
10 — shed-driver-pattern — add PATTERN-shed-driver, its seam test and the told-geometry binding
11 — batten-loop-arming — arm batten's until-stop loop and its interrupt policy
12 — batten-hand-back-notice — one orch notice per batten stop episode, shared with the row notices
13 — batten-drives-its-run — lyx batten run drives its own run through the sheddriver body
14 — driver-docs — overview and package docs follow the move

## Shared Decisions

### Decision: sheddriver-surface

`internal/sheddriver` imports no `lyxcwd`, `loomengine`, `loomshed` or `internal/*cli` package (D1).
It takes every path told and reaches recipe behaviour only through `Deps` and `TacklerDeps`.
Its exported surface is what the arming modules and loom's arbiter use: the records, the stop key, `Deps` and its optional seam groups, `Run` and its `Exit`, `DriveVerb`, the tackler call, consume and settings, the shuttle seam and causes, `AdmitRelaunch`, the config module and the resolvers.
Error and log prefixes are `sheddriver:`.

### Decision: moved-names-and-references

Cards that move whole files use `git mv` and keep each file's base name, except `internal/loomengine/tackler.go`, which becomes `tacklerrecords.go` so card 4's `tackler.go` can follow.
A name a moved file exports anew is declared as a `plan:` handle on the moving card.
Later cards never name a moved file by its new path; they reach moved code through those handles.

### Decision: override-representation

A module override inherits `shed.yaml` when empty: `""` for `driver_kind` and `tackler`, 0 for `tackler_timeout_min`, and null for `tackler_max_spawns`, which is a `*int` so a set 0 still turns the tackler off (D5).
The module templates ship these inherit values.

### Decision: test-budget-rows

Each card that adds or moves top-level tests into a package raises that package's row in `cmd/lyx/testdata/test-budget.yaml` in the same card.
`internal/sheddriver` goes 3, 9, 15, 19, 20 across cards 2, 4, 6, 7 and 10, and `internal/battencli` goes 80 and 81 across cards 3 and 8, card 13 replacing the test it deletes.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
