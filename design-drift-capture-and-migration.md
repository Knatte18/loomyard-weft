# Drift capture fails by name with no baseline, and a reinterpreted drift.yaml is migrated

`drift-capture-and-migration` · Task · bug, drift, loom, reconcile · running · Drift-Baseline

Two Drift faults around the baseline capture.

## Part 1: a capture exits 0 but writes no baseline (issue #41)

A loom run halted blocked at Drift-Await: the baseline capture ended failed because it exited 0 but wrote no file to `.lym/drift/baseline`. The capture log lives under `_lym/drift/logs/`.
The halt recurred on the next resume with the identical reason, so the retry the way forward names did not recover it; only a later manual resume got the row to done.
A capture that exits 0 yet produces no baseline either silently did nothing or wrote to the wrong place, and the operator gets no cause beyond the log path.
Provenance: task `report-significant-digits` on the `cs` hub.

Wanted:

- `lym drift capture` fails with a non-zero exit and a stated cause when it produced no baseline file.
- The capture's stated cause goes into the Drift-Await halt reason, not only the log path.
- Find why the capture produced no file on that run and why the first re-capture reproduced it.

## Part 2: moving Drift's dir base carried no migration

`c896481c` ("One base for a command's dir; a missing Drift dir fails by name"), deployed 2026-11-03 17:23, moved Drift's `dir` base from the anchor to the worktree root.
The `cs` hub's `drift.yaml` held `capture.dir: .` and `compare.dir: .`, written for the anchor base; nothing rewrote it, warned of it, or told `cs:orch`.
A capture of `report-significant-digits` then ran `dotnet run --project src/LymCsharp.Regression …` in the worktree root and failed `exit 1`: "The provided file path does not exist". The run dir existed, so the pre-spawn check did not fire, and the reason read only `exit 1`.
`cs:orch` found the cause by hand and set both dirs to `dotnet`. Part 1's halt may share this cause.

Wanted:

- A binary that changes the meaning of a config value carries its migration: `hubreconcile` rewrites, or flags with a way forward, each hub file whose value it reinterprets (here a relative Drift `dir` on an anchored hub) before the new binary runs a command from it.
- A hub's orch is told when such a reconcile changes or flags one of its files.
