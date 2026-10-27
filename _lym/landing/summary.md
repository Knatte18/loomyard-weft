# Darn runs Drift-Await after Drift-Baseline instead of a waiting capture

Darn now drives its baseline capture the way loom does.
Drift-Baseline records the scope and starts a detached capture, and a new Drift-Await row holds the run until the capture passed or the parent deferred it.
Before, darn's Drift-Baseline ran with `capture: wait` and blocked on the capture in its own process, so the two recipes drove the same capture two ways, and a `wait` mode existed only for darn.

## What changes

- `contracts/recipes/darn-recipe.yaml`: the row order is Preflight, Drift-Baseline, Drift-Await, Darn-Write, and so on. Drift-Await carries `entry_gate: drift.enabled` like the other Drift rows, so a hub that leaves regression off skips all three.
- `internal/shedrecipe`: the DriftBaseline entry accepts only `subject`; a `capture` key is refused with the way forward named.
- `internal/driftengine`: `Row.WaitCapture` and the code only it reached are gone, and Drift-Baseline no longer reads `Deps.Sleep`. `Deps.AwaitRow` and `VerbStatus.AwaitRow` are gone, and `CountedCapture(scope, capture, found)` counts a passed record only when `awaited_at` is set. `lym drift waive` and Drift-Verify judge a counted capture by `awaited_at` on every recipe.
- `internal/loomcli`, `internal/driftcli`: the wiring no longer computes or passes the await-row flag.
- `internal/reportengine`: darn's phase map puts Drift-Await in the setup phase, next to Drift-Baseline.
- Docs: the `doc.go` files of `driftengine`, `driftcli`, `shedrecipe`, `loomrecipe` and `reportengine`, the recipe's header comment and the darn paragraph of `docs/overview.md`.

A darn run that halts on its capture now halts at Drift-Await, as loom does, and `lym drift defer` and `lym drift capture` settle it the same way; neither verb needed a change.

## Design decisions

- Drift-Baseline keeps the breakaway fallback: when Windows refuses the detached start, the capture runs in its own process, as in loom. A held capture lock is left running, and the pass returns Done without waiting.
- `Deps.RunCapture` stays for that fallback, and `Deps.Sleep` stays for Drift-Await's wait.
- Darn's Drift-Await takes no config key, like loom's.
- There is no transition code. A darn run that passed Drift-Baseline before this change has no `awaited_at`, halts at Drift-Verify with no counted baseline, and is settled by hand with `lym drift waive`. The recipe's header says so. The window is a few minutes early in a darn run.

## To check by hand

- Nothing was run against a live darn run; the first darn task after landing exercises the new Drift-Baseline, Drift-Await handoff.
