# Baseline producer and numerical-drift gate

`baseline-drift-gate` · Note · feature, webster, loom

From `manifest/roadmap.md` Someday.
loomyard captures no before-state, so it cannot tell a pre-existing failure from one a task caused ([millhouse#1130](https://github.com/Knatte18/millhouse/issues/1130)).
In a numerical target repo (the drilling simulator) the failure to catch is drift, which an exit-code verdict like `plan.Verify`'s cannot express.

Agreed shape, not yet a design doc:
- A shared `Capture-Baseline` producer row right after `Preflight`, shared by reference the way `preflightshed` and `landingshed` share theirs, so the planner writes against the current numerical signature.
- The artifact lives in `.lyx/baseline/`, never `_lyx`: a numerical baseline is machine-bound (floating point, BLAS, compiler flags).
  Capture before the first edit, recompute after, compare, discard.
- Two seams, both owned by the target repo: `capture` (tree → artifact) and `compare` (before, after → verdict + findings, with per-channel tolerance).
  Loomyard owns the row, the seam shapes, and the artifact location.
- The gate rides `shuttleengine.GateSpec` on the `Webster` row; registering a validator name in `resolveGateSpec`'s switch is the unbuilt part.

Open:
- Whether compare runs once at Webster's exit (what `GateSpec` gives, since it fires at the Master run's finalize) or per batch (exact localization, a new batch-loop hook, N captures).
- A middle option: gate once, and localize by making the predicate of Webster's integration-stage `bisect` pluggable, at log₂(N) captures; bisect assumes monotone drift.
- A baseline tree whose capture itself fails should be recorded, not halt the run.