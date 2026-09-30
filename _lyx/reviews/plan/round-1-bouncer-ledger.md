---
round: 1
ledger:
  - key: weft-dirty-test-under-asserts-criterion-1
    rounds: [1]
    status: resolved
  - key: verify-integration-scope-too-narrow
    rounds: [1]
    status: resolved
  - key: impactsummary-missing-fabriccli-caller
    rounds: [1]
    status: resolved
  - key: planted-reuse-tag-must-be-lightweight
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so there was no prior ledger.
The reviewer approved the single-card plan with four findings.
The fixer resolved all four in the same round:

- The weft-dirty refuse-then-resume test now asserts all of acceptance criterion 1.
- The verify section runs the full `-tags integration ./...` suite.
- The ImpactSummary names both production callers of `Topology.Remove`.
- The same-tip reuse test plants a lightweight tag.

My judge pass found no new finding, and no entry remains open.
