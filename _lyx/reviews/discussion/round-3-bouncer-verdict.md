---
verdict: CONTINUE
rationale: "Round 3 carries BLOCKING finding F1 (origin reconcile moves a weft past D10's stop and the D7 move record), first raised in round 3 on a key no earlier ledger holds, so the rule gives CONTINUE and not CIRCLING; no round-3 finding maps to a key open in an earlier round, and no relabels were made."
---

Round 3 has five findings, down from thirteen.
F1 is BLOCKING and class `design`, on a new key, `origin-reconcile-moves-weft-past-d10`.
A BLOCKING finding blocks `CONVERGED` whatever its key's age, and the new key keeps the verdict off `CIRCLING`.

F2 (`checkout-refusal-set-target-untracked`) is LOW `design`.
F3 to F5 are LOW `consistency`.
None maps to a key open in an earlier round, so every earlier-round key is marked resolved in the ledger.

There is no `## Focus departures` section in the round-3 review, so nothing to ratify or reject.
No finding's class or severity was relabelled.
F1 cites no entry the declared changes list; it is weighed as the review's own attack-surface finding, and it stays BLOCKING.
