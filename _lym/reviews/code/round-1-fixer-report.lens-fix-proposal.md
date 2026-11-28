```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal check of the review proposals carried out as proposed (F1, F3, F4, plus the parts of F2 taken as proposed).

no findings

- **F1:** deleting the assertions breaks nothing. `ungatedTestStep` and `gatedNoGateStep` still pin which fragment Go chooses.
- **F3:** keeping the measurement calls in the writer's session does not conflict with item 2's per-package gate rule for subagents. A subagent that edits code still runs that gate for its own package.
- **F4:** narrowing the rule to the measurement calls leaves item 3's per-commit run in the ungated list and the verify gate in the gated one intact. I could not build a concrete case where either sentence gives a wrong result.
- **F2:** the stop-marker step it takes from the proposal resolves, since Step 3 of `darn-template-write.md` names the marker's path.

Further tool calls: none.
