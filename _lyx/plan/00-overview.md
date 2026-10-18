---
format: 5
approved: true
language: go
---

# Plan: Fast batten start, local-only fabric pairs and one Command producer

A batten start should reach Run-Shed in about the time its rows' own actions take.
The plan measures a start first, then gives fabric local-only pairs (a branch mark in gitrepo, `lyx fabric add --local`, and remove and cleanup paths that touch no origin), moves every per-transition status push onto fabric's detached push behind one spawn entry point, a try-lock with pending flags and a per-slug teardown lock, replaces batten's three switched rows with one generic `Command` producer and the `lyx` verbs its default commands call, and ends by measuring again and cutting what the numbers show.

## Card Index

1 — batten-start-before-numbers — record the before measurement of a batten start's rows
2 — gitrepo-local-branch-mark — the per-branch local mark and the push primitives' refusal of a marked branch
3 — fabric-push-skips-local-branches — fabric's pushes report a local branch as skipped
4 — fabric-add-local — `lyx fabric add --local`, and a plain add's adopt clears the mark
5 — fabric-remove-local-pair — a local pair's remove runs no origin step and drops its records
6 — fabric-cleanup-local-orphan — cleanup of a local pair's orphan weft branch pushes nothing
7 — status-push-spawn-entry — one spawn entry with the prime rule, slug roles, `_board` cwd and the local-mark check
8 — push-lock-pending-flags — the push lock is try-locked by children, with per-side pending flags
9 — pair-teardown-lock — a per-slug teardown lock between Remove and the pair's pushes
10 — status-seams-spawn-detached-push — batten's, loom's and darn's seams and batten cleanup spawn the push
11 — verifyrun-argument-quoting — literal-argument quoting beside the shell choice
12 — ide-seed-and-open-verbs — `lyx ide seed` and `lyx ide open`
13 — reed-window-verb — `lyx reed window`
14 — command-producer — the `Command` producer
15 — command-engine-replaces-switched-rows — the `Command` engine replaces the switched rows and engines
16 — batten-start-after-numbers — after numbers, the overhead they show cut, and the start-cost drive

## Shared Decisions

- A plain `lyx fabric add` adopts a local pair's kept warp branch (D3's adopt-clears rule names it), so card 4 lets the ordinary path adopt a warp branch carrying the mark instead of refusing `ErrBranchExists`; an unmarked existing branch still refuses as today.
- No test forks a real push child: `SpawnDetachedPush` returns `ErrSpawnUnderTest` in a test binary (card 7), and a test observing a spawn swaps the package's spawn variable.
- `lyx reed window` takes the worktree directory as its positional, since the `Command` row runs in the prime and the window attaches to the task worktree's session.
- The pending flag of a side holds that side's single-side push request, so a holder can respawn a side it does not push with the slugs of the push it serves (card 8).

## PATTERN changes

- `PATTERN-push-both-sides` — changes: Pushing run records from Go: the pair's code branch goes along, and a per-transition status push goes through fabric's detached push.
- `PATTERN-recipe-setting-lookup` — changes: Giving a recipe row a config value: a plain one is `module.key` via `Env.Setting`, an executable one `module.key` via the owner's told executable-key lookup over the template and board default layer, a review segment's resolved one a `SegmentSettings` field; never its own `Env` field.
- `PATTERN-command-row` — adds: Adding a side-action row to a recipe: a `Command` row over its config keys, its command read from the template and board default layer only, never a new engine.

## verify:

```
go build ./...
go test ./cmd/lyx ./internal/gitrepo ./internal/fabricengine ./internal/fabriccli ./internal/tendercli ./internal/lock ./internal/statuscommit ./internal/battencli ./internal/loomcli ./internal/cleanupcli ./internal/verifyrun ./internal/idecli ./internal/reedcli ./internal/shedadapters ./internal/shedrecipe ./internal/battenrecipe ./internal/battenshed ./internal/configreg ./internal/testkit/envkit ./internal/pattern ./internal/lyxcwd ./internal/loomrecipe
go test -tags integration ./cmd/lyx ./internal/gitrepo ./internal/fabricengine ./internal/fabriccli ./internal/tendercli ./internal/lock ./internal/statuscommit ./internal/battencli ./internal/loomcli ./internal/cleanupcli ./internal/verifyrun ./internal/idecli ./internal/reedcli ./internal/shedadapters ./internal/shedrecipe ./internal/battenrecipe ./internal/battenshed ./internal/configreg ./internal/testkit/envkit ./internal/pattern ./internal/lyxcwd ./internal/loomrecipe
```
