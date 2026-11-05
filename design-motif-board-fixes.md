# Motif runs: own profile recipe, a finished state on the board, and live status

`motif-board-fixes` · Task · bug, loom, board, batten, profiles · running · Drift-Baseline

Three motif faults from issues #58, #53 and #49, seen in hub Models-LYMHUB, beam `duplication-and-import-followups`, 2026-11-04. Fix all three.

## 1. The motif recipe names tapestry as its profile recipe (#58)

The motif recipe lists `profile_recipe: tapestry`; motif should not depend on tapestry's profile.

## 2. A finished motif reads as a plain draft (#53)

The motif task `knut-rheology-friction-pvt` ran Discussion to an approved decision record.
`lym batten status` reports state done, the record is stored on the entry (`lym board get --record` works), and the entry carries `discussed: true`.
But the board status went back to `draft`, so the board list and the rendered README show it like a task nobody has started; the operator read it as unfinished work.

Expected: a motif whose Discussion is approved shows as finished on the board, in the list, the `--text` listing and the README, rather than a plain draft.

## 3. The board status stays at Worktree-Create through Discussion (#49)

About 15 minutes after release, the same task's Discussion-Write session was live and writing the record, while:

- the board entry, and so the README, said `running · Worktree-Create`;
- `lym batten status` showed current_producer Run-Shed, history_length 5;
- `lym loom status` in the task worktree showed current_producer empty and history_length 0;
- `lym beam status` showed the run with an empty producer column.

Board history from Bolt's commits: draft, then four `running · Worktree-Create` commits, then `running · Discussion-Bouncer` 14 minutes later.
The status was never written for Seed-Child through Run-Shed, nor for Discussion-Write; the README renderer is not at fault.
A tapestry run in the same beam showed its inner producer, such as `running · Webster`.

Expected: the board status follows the batten rows past Worktree-Create and then the inner run's current producer, Discussion-Write here, and `lym loom status` names the producer in flight.
