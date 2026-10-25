---
verdict: CONTINUE
rationale: "Round 1 carries five design-class MEDIUM findings (F1, F2, F5, F6, F7), which are gating under the decision rule; no BLOCKING finding and no relabel."
---

The gating findings are F1 (usage.txt lacks window percentages), F2 (config.toml byte-identical not evidenced), F5 (BLOCKING counts not recomputable from committed data), F6 (workspace-write reads not confined) and F7 (grid.sh skips refused cells on rerun).
F3 and F4 are MEDIUM but class `scope`, and F8 to F12 are LOW, so none of them gates.
No finding is relabelled, and the review has no `## Focus departures` section.
