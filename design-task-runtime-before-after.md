# Measure whether tasks run faster since turn-savings, darn and tapestry apart

`task-runtime-before-after` · Task · research, report, loom · running · Darn-Write

## Question

Do tasks run faster today than before, and how much of any change comes from project `turn-savings` (the `agent-turns` changes and the batching work)?
The operator wants figures, for darn and for tapestry apart.

## Scope and cap

- Read recorded data only: no agent run, no new trial, no change to lym.
- Inputs: `lym report timing`, `lym report cost`, `lym gate report`, the commits of the landed runs, and the run records they point at.
  `lym report` runs from the hub's prime only, so run it with `env -C` on the prime; it writes nothing.
- Window: every landed run in the records, which `lym report timing --since 2026-09-01` shows starting 2026-10-27.
  Look once for older runs in the archive tags; if none hold phase data, say so and keep the window.
- A snapshot of about 274 landed runs was read on 2026-11-06: 111 darn before and 148 after the cut below, 7 tapestry before and 8 after.

## Periods

- Before: a run started before `f08df88b` (2026-11-01 15:04 UTC), the first `agent-turns` change.
- After: a run started after `cfad47e6` (2026-11-01 16:53 UTC), the last one.
- A run started between the two belongs to neither, and the report names how many.
- Inside After, also show the runs after the later changes apart where the sample allows: the batching hook (2026-11-02), its widening `119c1c05` (2026-11-04) and the Discussion exploration change `3085f451` (2026-11-05).
- Check the commits and the project's landed members against this list, and correct the cut if a member lies outside it.

## Measures

- The figure is active time: machine sleep left out, and every wait on someone else (escalation, halt, parent review, PR gate) and paused time left out.
  Read how `active_seconds` is computed first, so nothing is removed twice or kept by mistake, and show the waits apart.
- Give median and range beside every mean, with n beside every figure.
- Normalize by size: changed lines of each run from its landing commit, with time per 100 changed lines and a fixed-plus-per-line fit per period.
  A raw before-and-after comparison alone is not an answer, since the runs differ in size.

## Darn

- Compare by profile, and keep `docs` runs apart: a bench or measurement's write time is the time of what it measures, not of an agent's work, as with `codex-astra-fan-bench`.
- Where a run's profile is not recorded, derive it from its settings or mark it unknown; never fold unknown runs into a profile.

## Tapestry and a Sketch

A tapestry run that starts from a Sketch already holds its decision record, so it skips the Discussion phase.
Comparing its whole run with a run that had a Discussion would show a false saving.

- Mark a run Sketch-fed when it has no Discussion phase and a stored decision record; `lym report cost` carries the stored-record flag, and `slack-relay` is such a run in the snapshot.
- Whole-run comparison: only runs that ran Discussion, on both sides.
- Plan-onward comparison: Plan through Land for every tapestry run, Sketch-fed ones included, since that part is like for like.
- Show Discussion's own time as a phase of its own for the runs that had one.
- Sketch-fed runs are listed by name and never enter a mean of whole-run time.
- `cold-gate-fixer` shows no Discussion or Plan time: name it and leave it out of both comparisons, stating why.
- `creel-cross-hub-build` counts if it has landed when this task runs.
- The sample is small, so list each tapestry run with its phases and size; make no claim of significance.

## How much is P1

Time saved in a phase says nothing of its cause, so tell the phases apart.

- Per-phase tables for both periods, for each recipe.
- Phases `agent-turns` touched: write, plan and Webster.
  Phases it did not touch: setup, Finalize, Land.
- Name what else changed in the window and could move the figures either way, with its date: gate changes (fewer test binaries and `lym gate` keys), the drift rework, the arbiter, added review or recipe rows, a model change, and the number of runs in parallel competing for gate slots.
- Take gate time out of the phases where `lym gate report` allows it, and show whether the change remains.
- State plainly what the data supports and what it cannot separate; `docs/research/turns/turn-savings-measure-2026-11-04` already has the per-turn effect, so link the two rather than repeat it.

## Output

`docs/research/speed/task-runtime-before-after-2026-11-06/report.md`, with its scripts and tables beside it, and a line in `docs/research/README.md`.
The report opens with the answer: faster or not, by how much, for darn and for tapestry, and how much of that P1 can claim.
