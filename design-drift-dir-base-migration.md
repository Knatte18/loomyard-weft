# Moving Drift's dir base reinterpreted existing drift.yaml files with no migration

`drift-dir-base-migration` · Note · bug, drift, reconcile

The binary that moved Drift's `dir` base to the worktree root changed what an existing `drift.yaml` means and carried no migration, so a hub's capture silently moved to another directory.

## Seen

`c896481c` ("One base for a command's dir; a missing Drift dir fails by name") was deployed at 17:23 on 2026-11-03, while the Drift trial ran on the `cs` hub (anchor `lym-csharp/dotnet`).
Its message says: "A `drift.yaml` `dir` written relative to the anchor must be rewritten relative to the worktree root. For the `cs` hub, … `dir: .` now means the worktree root".

- `cs`'s `drift.yaml` held `capture.dir: .` and `compare.dir: .`, set an hour before for the anchor base; nothing rewrote it, warned of it, or told `cs:orch`.
- At 17:32:25 a capture of `report-significant-digits`, run from the task worktree's `dotnet/`, ran `dotnet run --project src/LymCsharp.Regression …` in the worktree root and failed `exit 1`: "The provided file path does not exist: src/LymCsharp.Regression."
  The run dir existed, so the new pre-spawn check did not fire, and the reason read only `exit 1`.
- `cs:orch` found the cause by hand and set both dirs to `dotnet`.

## Expected

- A binary that changes the meaning of a config value carries its migration: `hubreconcile` rewrites, or flags with a way forward, each hub file whose value it reinterprets (here a relative Drift `dir` on an anchored hub) before the new binary runs a command from it.
- A hub's orch is told when such a reconcile changes or flags one of its files.
