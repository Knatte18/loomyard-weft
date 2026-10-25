# Darn runs Drift-Await after Drift-Baseline instead of a waiting capture

`darn-drift-await-row` · Note · enhancement, shed, loom

## Seen (operator, 2026-10-25)

Darn's recipe runs Drift-Baseline with `capture: wait`, so the baseline row both records the scope and blocks on the capture in its own process, while loom splits the same work into Drift-Baseline, which starts a detached capture, and Drift-Await, which holds the run until the capture passed or the parent deferred it.
The two recipes therefore drive the same capture two ways, and `internal/shedrecipe/entries_drift.go` and `internal/driftengine` carry a `wait` capture mode that only darn uses.

## Idea (operator, 2026-10-25)

- Darn runs Drift-Baseline, then Drift-Await at once, then Darn-Write, the same two rows loom runs, with Drift-Await carrying `entry_gate: drift.enabled` like the other Drift rows.
- Drift-Baseline always detaches its capture; the `capture` config key and its `wait` mode go from `shedrecipe` and `driftengine`.
