---
format: 5
approved: false
language: go
---

# Plan: run-process-ownership

Every process lyx starts for a run joins one owner record, `internal/runproc`, keyed by the run's slug or the hub scope, so `lyx tender status` lists it and every verb that ends or tears down a run ends all of it through `runproc.End`.
The cards build the record and its end sequence, register the detached and long-lived run processes, make detached pushes and board syncs exit under a pause and every lift spawn what they owe, end owned processes from pair teardown, `lyx fabric remove`, `lyx reed down` and the owned-elsewhere pauses, add `lyx tender reap` for orphans, and settle hibernate confirmation and shuttle's waiting turn end over shell-only background work.

## Card Index

1 — proc-stat-reads — proc reads every process's parent, group, start time and zombie state, and signals a pid or a group
2 — runproc-owner-record — internal/runproc: one record file per run process, and runproc.End
3 — register-detached-spawns — detached pushes and board syncs register under the owner D2 names
4 — register-run-roles — the Go driver, the step loops and a seeded batten register themselves under their slug
5 — gate-waiting-marker — the network gate leaves a waiting marker, and one process may pass every entry
6 — deferrable-gate — the gate of a detached push or board-sync child defers instead of waiting
7 — detached-children-defer — detached pushes and board syncs exit under a pause, flags and commits kept
8 — lift-spawns-owed — every lift spawns the pushes and the board sync a pause deferred
9 — teardown-ends-owned — pair teardown ends the pair's owned processes before its loop and session
10 — teardown-skip-readings — teardown skips the driver checks of a run with nothing left to do and waits for the run lock first
11 — fabric-remove-reconcile — lyx fabric remove reconciles first and removes an ended or foreign run locally
12 — reed-down-ends-owned — lyx reed down ends a pair's owned processes first and refuses a running unpaused run
13 — owned-elsewhere-end — an owned-elsewhere pause ends the run's processes on the machine it left
14 — tender-status-processes — lyx tender status lists owned processes and orphans
15 — tender-reap — lyx tender reap ends the orphans of one fresh reading
16 — hibernate-confirm-background — hibernate confirmation settles shell-only background work and waits out forks
17 — shuttle-shell-only-waiting — shell-only background work never holds a row its outputs settle

## Shared Decisions

- The record directory is `fabricengine.RunProcDir(hub)`, `<HubScratchDir>/runproc`, declared beside `HubScratchDir` and handed to `runproc`, which derives no path (`PATTERN-told-geometry`, `PATTERN-lyxdirs-single-declarer`).
- The hub-scope owner is `runproc.HubOwner`, `_board`, a name the slug check refuses as reserved hub geometry.
- Roles are `runproc` constants: `driver`, `loop`, `batten`, `push`, `sync`.
- `shedverbs` takes a `LoopSpec.OnStart` hook instead of importing `runproc`, so its import allowlist stays as it is.
- A detached child of an interrupted CLI call is not tied to that call, departing from the board entry's "what should hold" as D2 records: the owner record and the end verbs reach it instead.
- `End` is called with the caller's own pid everywhere, so batten's Worktree-Teardown and its owner probe spare batten and its ancestors.
- D6's lift spawns a board sync "when Bolt is ahead of its upstream"; since a sync deferred at its start exits before it commits, the lift reads "ahead" as ahead or holding uncommitted board changes, the reading that lets the lift spawn what a deferred sync owes (card 8).
- The three owned-elsewhere sites share one end: `tendercli.PauseOwnedElsewhere` already serves batten's probe and `lyx orch start`, so it ends the run after the pause; the hibernate wake gains a seam filled with the same `tendercli.EndOwnedElsewhere`.

## PATTERN changes

- `PATTERN-run-process-owner` — adds: Starting a detached or long-lived process for a run: register it in `internal/runproc`'s owner record; ending a run's processes: only via `runproc.End`.
- `PATTERN-pair-teardown` — changes: Every teardown of a pair goes through `internal/pairteardown`, which ends the pair's owned processes and then its reed session before any worktree is removed; its driver checks are skipped for a pair that is paused, ended, owned by another machine, or done locally with batten no longer running, never its run-lock wait while the task worktree is present, which then runs before the owned processes are ended.
- `PATTERN-leaf-packages` — changes: adds `internal/runproc`, importing only the standard library, `internal/proc`, `internal/state` and `internal/logger`.
