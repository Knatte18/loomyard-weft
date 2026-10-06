---
format: 5
approved: true
language: go
---

# Plan: bugfix-fabric

Fixes the three bugs the board entry `bugfix-fabric` reports, each with a regression test that drives the real CLI wiring on a `hubforge` hub in the `integration` tier (D8), test files named by path (D9).
`weft-lock-dir-drops-parent`: every weft worktree fabric creates gets its `.weft/` lock directory at birth, reconcile heals a missing one, and `ReadOriginFor` refuses a record without one instead of answering "no record" (D1–D3).
`fabric-add-adopt-remote`: one weft-branch resolver (local, then origin, then none) is shared by `checkout`, `reconcile` and `add`, and `add` adopts a live pair's warp branch from origin (D5–D7).
`config-reaches-parent`: Finalize reads the task's per-worktree config changes since its fork point and queues one orch notice at Done, never carrying them (D4).
No new verbs, flags, config keys or envelope keys.

## Card Index

1 — weft-lock-dir — create `.weft/` at every weft worktree birth, heal it in reconcile, and make `ReadOriginFor` refuse a record without it
2 — weft-branch-resolver-checkout — add the local/origin/none weft-branch resolver and use it in `checkout`
3 — reconcile-adopts-origin-weft — `reconcile` adopts an origin weft through `weft_recreated` before its raw and unmanaged branches
4 — add-adopts-live-pair — `add`'s live-pair rule: adopt origin weft and warp, fast-forward a behind local weft, keep an existing origin record
5 — read-config-changes — read-only fabricengine diff of a task's per-worktree config files since its fork point
6 — finalize-config-notice — Finalize queues one orch notice for those changes at Done, wired through loomcli

## Shared Decisions

- All git on warp or weft stays in `internal/fabricengine` through `gitexec.Run`; no new `//gitexec:raw` site (`PATTERN-fabric-git`, `PATTERN-gitexec-checked-call`).
- Every branch creation, origin-tracked branch creation and fast-forward appends to the verb's `*Mutations` record; rollback of an origin-tracked branch goes through `deleteBranch`, and no origin branch is ever deleted or force-pushed by these changes (`PATTERN-mutation-record`, `PATTERN-fabric-destruction-chokepoint`, `PATTERN-no-denied-recovery`).
- A weft repo with no `origin` remote skips every origin step; an unreachable or failing origin is an error naming `origin`, never a silent fork.
- `landingshed` names neither fabric side and derives no path; it receives the diff and the notice queue as told closures (`PATTERN-told-geometry`, `PATTERN-fabric-vocabulary`, `PATTERN-orch-pane-single-writer`).
- `internal/fabriccli/adoptremote_integration_test.go` is created by card 2 and extended by cards 3 and 4; origin-only branches are pushed into `h.WarpBare`/`h.WeftBare` from scratch clones, and an unreachable origin is simulated by pointing the weft repo's `origin` URL at a missing path.

## verify:

```
go build ./... && go vet -tags llm ./internal/fabricengine/... ./internal/fabriccli/... ./internal/hubgeom/... ./internal/landingshed/... ./internal/loomcli/... && go test ./... && go test -tags integration ./internal/fabricengine/... ./internal/fabriccli/... ./internal/hubgeom/... ./internal/landingshed/... ./internal/loomcli/... && go test -tags tmux ./internal/fabricengine/... ./internal/fabriccli/... ./internal/hubgeom/... ./internal/landingshed/... ./internal/loomcli/...
```
