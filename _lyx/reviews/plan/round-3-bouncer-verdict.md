---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) is the same fixtures-untargeted-by-card-5 gap as key tighter-read-rules-fixtures-untargeted, open in rounds 1 and 2 and again in round 3, now on the CIRCLING replay path (TestBouncer_CirclingGuard) instead of the CONVERGED path."
---

Mapping: F1 maps to `tighter-read-rules-fixtures-untargeted`.
The key names card 5's tightened ledger and review read rules breaking fixtures the card does not target.
Round 3 hits two rows of another test under the same rules, so the key is reopened at BLOCKING rather than a new key.

The facts list the key as open in rounds 1 and 2, and F1 is gating (`design`) at BLOCKING, so the decision rule gives `CIRCLING`.

Focus departure ratified: F1 follows the guard's ledger read beyond CONVERGED settles.
Card 5's own Intent claims every round-2-or-later ledger a settle reads keeps its previous keys, so checking that claim against replay-path tests is within the rubric.

No relabel of any finding.
Directive verdicts 1 and 2 found no defect and are accepted.
