# Darn runs Drift-Await after Drift-Baseline instead of a waiting capture

`darn-drift-await-row` · Task · enhancement, shed, loom · running · Code-Burler-Review

## Seen (operator, 2026-10-25)

Darn's recipe runs Drift-Baseline with `capture: wait`, so the baseline row both records the scope and blocks on the capture in its own process, while loom splits the same work into Drift-Baseline, which starts a detached capture, and Drift-Await, which holds the run until the capture passed or the parent deferred it.
The two recipes therefore drive the same capture two ways, and `internal/shedrecipe/entries_drift.go` and `internal/driftengine` carry a `wait` capture mode that only darn uses.

## Change (operator, 2026-10-25)

- Darn runs Drift-Baseline, then Drift-Await at once, then Darn-Write, the same two rows loom runs, with Drift-Await carrying `entry_gate: drift.enabled` like the other Drift rows.
- Drift-Baseline always detaches its capture; the `capture` config key and its `wait` mode go from `shedrecipe` and `driftengine`, with `WaitCapture` and the code only it uses.
- With an await row in both recipes, the no-await-row branch of `CountedCapture` (its `awaitRow` parameter) goes too, unless another caller still needs it.
- A darn run that halts on its capture now halts at Drift-Await, as loom does, and `lym drift defer` settles it the same way.

## Decisions (operator, 2026-10-27)

- No transition code for a darn run that sits between its drift rows when this change lands and the new `lym` takes over: the window is a few minutes early in a darn run, and the orch settles a run it hits by hand.
