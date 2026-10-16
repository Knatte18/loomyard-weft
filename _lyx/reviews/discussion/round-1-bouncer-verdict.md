---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING design findings, F1 (D4 drops batten's tamper snapshot with no stated bound) and F2 (D5's value-equality Migrate rewrite is permanent and reaches run_settings); no relabels, and these fixes are new text no fresh reviewer has seen."
---

The latest round has 2 BLOCKING findings, both class `design`, so the decision rule gives `CONTINUE`.

- F1 and F2 are BLOCKING and gating. Neither is relabeled.
- F6 is class `design` but LOW, so it is not gating. F3 and F4 are MEDIUM `consistency` and F8 is LOW `decision`, none gating.
- No finding is relabeled in class or severity.
- The review has no `## Focus departures` section, so there is nothing to ratify or reject.
- No ledger key recurs or was open earlier; this is the first round.
- No pattern entry is cited in a finding that would conflict with the rule above.
