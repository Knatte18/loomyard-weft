# Weaver: attempt timeout, summary deviations and verify gate spans

The Weaver benchmark in `docs/research/weaver/bench-2026-11-27/report.md` found that Weaver can replace Webster
as tapestry's implementer, but named changes to make first.
This makes them.

- **Attempt timeout.** A new `weaver.attempt_timeout_min` key, 90 by default, bounds each batch attempt's strand.
  Attempts previously ran under `shuttle.run_timeout_min` (30 minutes), gate waits and gate fixers included,
  and the benchmark's first calibration batch hit that limit with three of four cards committed.
  A non-positive value is refused.
- **Summary.** `summary.md` now names, under each batch, the files changed outside its targets
  and its `Pattern-Departure:` trailers, or `none`.
- **Gate spans.** The Weaver row records `gate_spans` for the plan-level verify under the gate name `verify`,
  on done and stuck results.
- **Tests.** They now cover a strand that dies before its report, the ladder's third rung,
  further resume cases and the spans of a plan-level verify that passes or stops the row.

## Design decisions

- The timeout is a key of Weaver's own, sized to a batch; `shuttle.run_timeout_min` is unchanged.
- The bound counts from the strand's spawn.
  For a strand that a crashed call left, it starts again at the attach.
- Each run of the verify, reruns included, is one `gate` span.
  Each verify fixer that spawned is one `rework` span, however its run ends.
- A fixer that a later call attaches to gets no `rework` span, and spans cover only the current call's verify.

## To check

- This change holds no new benchmark run, so the 90-minute default is not measured against a live batch.

## Merge danger

**Door:** two-way
**Blast radius:** weaver
