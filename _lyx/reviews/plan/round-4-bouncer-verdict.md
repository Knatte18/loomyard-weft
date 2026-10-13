---
verdict: CONTINUE
rationale: "Round 4 carries BLOCKING finding F1 (design, card 9's decision-read guard rows unreachable and handedLockStep's verdict should be keep), mapped to the new key decision-read-guard-rows-unreachable, so the loop continues; F2 is consistency, non-gating, on the recurring recovery-branches-unreached-by-tests key; no departures or relabels."
---

F1 is BLOCKING and design-class, so the loop continues regardless of key history.
It is a different defect from `d11-audit-table-incomplete`: the D11 table now covers the sweep, and F1 faults one card's guard rows and one verdict.
That key and the two round-3 keys (`repair-session-name-keep-verdict-false`, `resize-pins-keep-ignores-fallback-box`) are not raised again, so they are resolved.
F1 therefore sits on a key first raised this round, and the verdict is `CONTINUE`, not `CIRCLING`.

F2 is consistency, so it is not gating.
It is the same reachability-of-recovery family as `recovery-branches-unreached-by-tests`, now on the strand-read test, and stays open.

No finding was relabeled and the review carries no `## Focus departures` section.
