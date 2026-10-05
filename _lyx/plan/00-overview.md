---
format: 5
approved: false
language: go
---

# Plan: board.yaml is hub-wide, read and written at the board dir

`board.yaml` describes the one board per hub, but today every `lyx board` verb loads it from the worktree it runs in, so a pair created before a vocabulary change re-renders README and validates labels against a stale copy.
This plan makes `board` hub-wide beside `fabric`, declared once in `configreg`.
Every board-config reader then loads `<hub>/_board/_lyx/config/board.yaml`.
`lyx fabric clone`/`reconcile` seed and reconcile the hub file, seeding from the prime's copy, and commit it in `_board`.
`lyx config` edits a hub-wide module at the board dir and commits it there, and `lyx config reconcile` retires per-worktree copies the hub file subsumes.
Deployment note for the PR description: after the landed binary is deployed, run one `lyx fabric reconcile` per hub, since board verbs refuse until the hub `board.yaml` exists.

## Card Index

1 — configreg-hub-wide — add `Module.HubWide`, set for `fabric` and `board`
2 — bolt-commit-written — `Bolt.CommitWritten` writes and commits named paths under the board write lock
3 — hub-wide-reconcile — `ReconcileHubWideAt` replaces the fabric-only reconcile, seeds `board.yaml` from the prime, and `fabric reconcile` commits it in `_board`
4 — retire-worktree-copy — `ReconcileAll` skips hub-wide modules by the registry field and retires subsumed per-worktree copies
5 — board-readers-hub-file — `lyx board`, loom's landing board and `lyx ide` menu load the hub `board.yaml`
6 — configengine-staged-edit — `EditPath` and `StagingFile` so an editor can work on an untracked staging copy
7 — config-cli-hub-wide — `lyx config` edits, sets, prints and menus hub-wide modules at the board dir, commits in `_board`, and PATTERN gains `PATTERN-hub-wide-config`

## Shared Decisions

- Hub-wide handling always branches on `configreg.Module.HubWide`; no site compares a module name to `"fabric"` or `"board"` for it.
- Hub, board dir and prime are resolved only in CLI packages (`fabriccli`, `configcli`, `boardcli`, `loomcli`, and `ideengine.Menu`, which already takes a resolved `lyxcwd.Location`); `configsync` and `boardengine` are told absolute paths.
- Commits of hub-wide config go through `Bolt.CommitWritten`, which holds the board write lock across the write and a commit scoped to the written paths, then `Bolt.Push`.
  `CloneAndWire` keeps its existing whole-repo `Bolt.Commit` instead: on a fresh clone nothing else writes `_board`, and that commit already carries every file the clone wrote.
- The interactive editor for a hub-wide module works on a staging copy under the worktree's `.lyx`, and only the copy into `_board`, its validation and its commit run under the board write lock.
  D5 requires that the write and the commit happen under the lock; the hub file is written only at that copy step, so the lock is never held while an operator's editor is open, which would otherwise stall every board write from running agents.
- The prime's base dir for seeding is `<hub>/<PrimeName>/<AnchorRel>`, reading its `_lyx` through the warp junction; clone passes `res.WeftBase`.

## verify:

```bash
go build ./... && go test ./... && go test -tags integration ./... && go vet -tags smoke ./...
```
