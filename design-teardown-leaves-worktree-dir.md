# Motif run leaves its worktree directory behind, and the leftover then blocks board and cancel

`teardown-leaves-worktree-dir` · Note · bug, batten, fabric, board

Imported from [issue #54](https://github.com/Knatte18/loomyard/issues/54).

## What happened

Models hub, beam `duplication-and-import-followups` (P2), 2026-11-04.
The motif task `knut-rheology-friction-pvt` (T2) ran to the end: Run-Shed done, Land done, Worktree-Teardown done (18:04Z per `lym batten status`).
The directory `Models-LYMHUB/knut-rheology-friction-pvt` was left behind. It was no git worktree any more and held only `obj/` and `bin/` output (10 files).

Afterwards:

- `lym board upsert '{"slug":"knut-rheology-friction-pvt","project":"knut-rheology"}'` was refused: "entry has a pair, ... end the pair first with lym batten cancel knut-rheology-friction-pvt --remove".
  This held after `lym batten cleanup --apply` removed the done run's records.
- `lym batten cancel knut-rheology-friction-pvt --remove` failed: "could not read the task worktree of knut-rheology-friction-pvt: not a git repository".
- The beam orch could not move its finished motif to another project, so the beam could not be emptied for its landing.

Only a manual `rm -rf` of the directory by the operator unblocked the board write.
The same leftover exists for many earlier runs in that hub (drift-numerical-baseline, flat-test-namespaces, tool-joint-hydraulics, ...).

## Expected

- Worktree-Teardown (and `lym fabric remove`) removes the whole directory, ignored build output included, and fails loudly if it cannot.
- The board's "has a pair" check and `batten cancel` treat a directory that is no git worktree as no pair, or name the exact cleanup.
- A motif, which lands no code, should not need a worktree teardown to finish before its entry can move.

Related: #21 (fabric remove leaves obj dirs).