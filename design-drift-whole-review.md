# Review Drift as a whole and redesign it from the trials

`drift-whole-review` · Note · research, drift, stencil, loom

## Seen (operator and hub orch, 2026-11-03)

The two Drift trials on the `cs` hub found many faults, each fixed on its own: `docs/research/drift-csharp-trial-2026-11-03.md` and the second trial's report under `docs/research/`.
The mechanics held: capture, compare, the anchor and the scoped gates.
Most faults sat where an agent judges (what Drift-Baseline skips or pre-accepts, what Drift-Verify accepts) or in gaps in the flow (no compare after review, no verdict in the pull request, a config base moved without migration).
No one has looked at Drift as a whole.

A skip list cannot carry an intended change: skipping the tests a change drifts on purpose hides any regression in them, and a starter's `run_settings` scope says what to skip, never what drift to expect.

## Plan (operator, 2026-11-03)

1. Wait until task `drift-baseline-preaccepts` and the second `cs` trial have landed and its report is committed.
2. A review, run as darn with profile `docs`, maps every Drift row, verb, stencil and config key; what each agent decides and what code decides; and every trial fault against that map.
   Its subject is Drift's own behaviour with no starter override: a `run_settings` drift scope bypasses the agents under review, so it is no answer to their faults.
   Its report proposes one reworked design: which judgments move into code, which stay with an agent and under what rules, and what the flow must guarantee.
3. The operator decides the design; a tapestry may then build much of it at once.

## Starting point for the design (operator and hub orch, 2026-11-03)

Judgment stays, but bounded, so an agent never decides freely whether a drift is fine:

- The expectation is written before the code, from the task alone: Drift-Baseline states which cases and which lines change, and how, such as "the report line goes to 4 significant digits, nothing else".
  It resolves no contradiction in the task and pre-accepts nothing.
- No test is skipped; everything is compared.
  Code checks where the drift falls: a drift outside the expected cases and lines is a regression, decided by code, not by an agent.
- Inside the expected cases and lines, an agent other than the writer judges whether the new values match the expectation.

So judgment rules on whether a value is right, while whether a drift may occur at all was fixed before the code existed.
A test expected to drift that drifts on another line, or to another value, is caught.
