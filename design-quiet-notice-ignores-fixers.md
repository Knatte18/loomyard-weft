# Quiet notice ignores gate fixers and the gate run

`quiet-notice-ignores-fixers` · Task · bug, dobby · running · Worktree-Create

## Seen (2026-11-05)

Dobby sent two quiet notices ("no agent activity readable") for darn runs that were working:

- `locks-out-of-tracked-trees` (#668): a gate-fixer ran 06:05–06:18Z, the writer finished at 06:19Z and the darn-write gate ran tier0–tier2 until 06:26:14Z. The notice came at 06:26Z, in the gap between the writer's shuttle ending and the next row starting; the writer strand was gone.
- `motif-board-fixes` (#669): the writer's strand sat idle while two gate-fixer sessions ran in their own windows (07:05–07:12 and 07:13–07:26 local). The notice counted only the writer's strand.

Each notice cost the orch a stop investigation that found nothing to do.

## Expected

The quiet check counts every session and process of the run as activity: gate-fixer sessions, a running `lym gate test` of the run, and the row hand-over after a writer ends.
A quiet notice fires only when none of them shows activity for the threshold.
