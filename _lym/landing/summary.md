# Darn runs Drift-Await after Drift-Baseline instead of a waiting capture

Darn's recipe ran Drift-Baseline with `capture: wait`, so one row both recorded the regression scope and blocked on the capture in its own process.
Loom splits that work between Drift-Baseline, which starts a detached capture, and Drift-Await, which holds the run until the capture passed or the parent deferred it.
Darn now runs the same two rows: Preflight, Drift-Baseline, Drift-Await, then Darn-Write, with Drift-Await carrying `entry_gate: drift.enabled` like the other Drift rows.
A darn run that halts on its capture now halts at Drift-Await, where `lym drift capture` and `lym drift defer` settle it as they do in loom; neither verb changed, since both already admitted a blocked Drift-Await on any recipe.

## What goes

- The DriftBaseline registry entry no longer accepts a `capture` key; a row that still sets one is refused as an unknown key, naming the way forward.
- `driftengine.Row.WaitCapture` and the code only it reached: Drift-Baseline no longer waits on a held capture lock, which it now always leaves running, and no longer reads the `Sleep` seam.
  The in-process `RunCapture` stays, for the fallback when Windows refuses the detached start breakaway.
- `Deps.AwaitRow`, `VerbStatus.AwaitRow` and `CountedCapture`'s `awaitRow` parameter: with an await row in both recipes, a counted capture is always a passed record whose scope record holds `awaited_at`.
  `loomcli`'s `driftDeps` loses the flag, and `driftcli`'s `verbStatus` no longer reads the run's seed, which it read only to set it.

## Design decisions

- The run report's darn phase table maps Drift-Await to the setup phase, beside Drift-Baseline, so darn time spent waiting on the capture counts as setup rather than `other`.
  The decision record assumed the phase map needed no darn change; it did, because darn keeps its own table.
  The darn timing fixture now lists the three Drift rows where they run, with the review-producer change moved after Darn-Write so it still tests what it did.
- No transition code: per the board entry's 2026-10-27 operator decision, a darn run parked between its drift rows when the new `lym` takes over is settled by hand, and the recipe header says so.

## Docs

`internal/driftengine/doc.go`, `internal/shedrecipe/doc.go`, `internal/loomrecipe/doc.go`, `internal/driftcli/doc.go`, the darn recipe's header and the darn and regression-row paragraphs of `docs/overview.md` describe the new row order and drop the wait mode.
