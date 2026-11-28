# Weaver: attempt timeout, summary deviations and verify gate spans

The Weaver bench of 2026-11-27 (`docs/research/weaver/bench-2026-11-27/report.md`) found four things to fix before Weaver can stand in for Webster as tapestry's implementer.
This change makes all four.

## What changes

- **A batch attempt has its own timeout.**
  A new key, `weaver.attempt_timeout_min` (default 90), bounds a batch attempt's strand.
  Until now the spec carried no timeout, so an attempt fell back to `shuttle.run_timeout_min` (30).
  The attempt's card and batch gate runs, and their in-session fixes, count against that bound.
  On the bench, the first calibration batch hit it with three of its four cards committed.
  The bound is set both where an attempt spawns (`runBatch`) and where a crashed call's attempt is probed and attached (`resumeAttempt`), since an attach takes its deadline from the spec.
  `LoadConfig` refuses a non-positive value, naming the key.
- **`summary.md` names each batch's deviations and departures.**
  Under each batch line it now lists the files the batch changed outside its targets, which is the scope guard's `deviations` list on the digest, and its `Pattern-Departure:` trailers, which is the digest's `departures` list.
  Either list reads `none` when empty.
  Webster's Master writes the same two lists into its summary.
  Before this change Weaver kept them only in `state.json`.
- **The Weaver row records `gate_spans` for the plan-level verify.**
  `RunResult` gains `VerifySpans`.
  Each run of the verify, the rerun of a failure included, adds one `gate` span.
  Each verify fixer adds one `rework` span, from its spawn to its end.
  `WeaverProducer` maps them onto the row's history entry as spans of one entry named `verify`, through the same `gateSpansOf` mapping Webster's row uses.
  Webster's row records its verify under the entry name `verify` too.
  A done result and a stuck result both carry the spans.
  A call that never reached the verify carries none.
- **Tests cover the paths the bench did not reach:**
  - a batch recorded done on the third rung after two failures, with the third prompt told the second attempt's failure;
  - a lost strand whose pane dies before it writes its report, beside the existing wait-error loss, both retried once on their rung and then climbed;
  - a resume that continues on the third rung after a failed second-rung attempt, and one that retries a lost last-rung attempt on that rung;
  - the spans of a failed plan-level verify that a fixer clears, and of one that exhausts the fixer ladder.

## Design decisions

- The default of 90 minutes is more than twice the slowest batch the bench measured: 39m44s, under a load average of up to 45 on a 12-core machine.
  Webster's `master_timeout_min` (480) bounds a whole run, so it was no model for a per-batch bound.
- A fixer's span is `rework`, not `gate`, so a reader can tell verify time from fix time the same way Webster's spans separate a gate run from the re-prompt that follows it.
- A verify fixer that a crashed call left running, and that the next call attaches to while it settles the crash, adds no span.
  Only the verify runs and fixers that `finishRun` starts are timed.
- The deviations are the scope guard's file list as recorded on the digest.
  The summary does not recompute them.

## Merge danger

**Door:** two-way
**Blast radius:** weaver
