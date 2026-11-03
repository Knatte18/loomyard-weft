# Review Drift as a whole and redesign it from the trials

`drift-whole-review` · Note · research, drift, stencil, loom

## Seen (operator and hub orch, 2026-11-03)

The two Drift trials on the `cs` hub found many faults, each fixed on its own: `docs/research/drift-csharp-trial-2026-11-03.md` and the second trial's report under `docs/research/`.
The mechanics held: capture, compare, the anchor and the scoped gates.
Most faults sat where an agent judges (what Drift-Baseline skips or pre-accepts, what Drift-Verify accepts) or in gaps in the flow (no compare after review, no verdict in the pull request, a config base moved without migration).
No one has looked at Drift as a whole.

## Plan (operator, 2026-11-03)

1. Wait until task `drift-baseline-preaccepts` and the second `cs` trial have landed and its report is committed.
2. A review, run as darn with profile `docs`, maps every Drift row, verb, stencil and config key; what each agent decides and what code decides; and every trial fault against that map.
   Its report proposes one reworked design: which judgments move into code, which stay with an agent and under what rules, and what the flow must guarantee.
3. The operator decides the design; a tapestry may then build much of it at once.
