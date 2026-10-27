# Darn runs Drift-Await after Drift-Baseline instead of a waiting capture

Darn drove the baseline capture differently from loom: its Drift-Baseline row used `capture: wait` and blocked on the capture in its own process, while loom starts a detached capture in Drift-Baseline and holds the run at a separate Drift-Await row.
Darn now runs the same two rows: Preflight, Drift-Baseline, Drift-Await, Darn-Write, with Drift-Await carrying `entry_gate: drift.enabled` like the other Drift rows.
A darn run that halts on its capture now halts at Drift-Await, as loom does, and `lym drift defer` settles it the same way.

With one capture path left, the `wait` mode and the code only it used are removed.

## Changes

- `contracts/recipes/darn-recipe.yaml`: Drift-Baseline drops `capture: wait` and hands off to the new Drift-Await row, which hands off to Darn-Write.
  The header comment is updated to match.
- `internal/shedrecipe`: the `DriftBaseline` entry no longer accepts a `capture` key.
  A row that sets it is refused by the existing unknown-key refusal, which still names the way forward.
- `internal/driftengine`: `Row.WaitCapture`, `Deps.AwaitRow` and `VerbStatus.AwaitRow` are gone, and so is the `awaitRow` parameter of `CountedCapture`.
  A counted capture is now always a passed record with `awaited_at` set, so `lym drift waive` and Drift-Verify judge it that way on every recipe.
- `internal/loomcli`, `internal/driftcli`: the wiring no longer computes or passes the await-row flag.
- `internal/reportengine`: Drift-Await is placed in darn's setup phase in the run report.
- Docs: the `doc.go` files of `driftengine`, `shedrecipe`, `driftcli`, `loomrecipe` and `reportengine`, and the darn paragraph of `docs/overview.md`.
- Tests that pinned darn's row list, edges and row indexes, or the removed modes, follow the new recipe; the removed modes' test rows are deleted.

## Design decisions

- Drift-Baseline always detaches its capture.
  When Windows refuses the detached start breakaway, it still runs the capture in its own process, as loom's row does; a held capture lock is left running.
  So `Deps.RunCapture` stays, and so does `Deps.Sleep`, which Drift-Await still uses; Drift-Baseline no longer reads `Sleep`.
- Both recipes with Drift rows now have an await row, so the no-await-row branch of `CountedCapture` had no caller and was removed rather than kept.
- Darn's Drift-Await takes no config key, the same as loom's.
- There is no transition code, per the operator's decision.
  A darn run that passed Drift-Baseline before this lands has no `awaited_at`, halts at Drift-Verify with no counted baseline, and is settled by hand with `lym drift waive`.
  A run parked on Drift-Baseline mid-capture finds the lock held and moves on without waiting.
  The recipe header says so.

## To check by hand

Nothing was run against a live darn run.
After landing, confirm that a darn run with `drift.enabled` halts at Drift-Await on a failing capture, that `lym drift defer` and `lym drift capture` settle it, and that every Drift row is skipped with `drift.enabled` off.
