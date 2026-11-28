# Weaver gets a batch timeout of its own, names deviations in its summary, records gate spans and is tested on the paths its bench missed

`weaver-bench-fixes` · Task · enhancement, weaver, shed, shuttle · running · Code-Burler-Review

## Why

The Weaver bench of 2026-11-27 (`docs/research/weaver/bench-2026-11-27/report.md`) found Weaver can replace Webster as tapestry's implementer,
but named four changes to make first (operator, 2026-11-28: fix them, then bench again).

## Changes

1. A Weaver attempt gets a timeout of its own, sized to a batch.
   Today an attempt runs under `shuttle.run_timeout_min` (30 minutes), which its gate waits and gate fixers count against;
   the bench's first calibration batch hit it with three of four cards committed, and the retry climbed to Opus.
2. Weaver's `summary.md` names each batch's deviations and departures, as Webster's summary does.
   Today Weaver records the deviations only in `state.json`, and its summary lists attempt counts.
3. The Weaver row records `gate_spans` for the plan-level verify, as the Webster rows do.
4. Tests pin the paths the bench did not reach: a lost strand, a resume, a failed plan-level verify and the ladder's third rung.
