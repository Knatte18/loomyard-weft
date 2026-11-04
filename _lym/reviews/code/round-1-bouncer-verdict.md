---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (timing-dependent capture-failure subtest, no test for the verb's non-zero exit) and design-class MEDIUM findings F3 (commandReport cuts the final log line) and F4 (reconcile notifier untested); no relabel was made."
---

Round 1 has no prior ledger, so no key recurs and none was open earlier.

Gating findings: F1 (design, BLOCKING), F3 (design, MEDIUM), F4 (design, MEDIUM).
F2 is class `decision` at MEDIUM: not gating by class, and it is deferred to the run's parent.
F5, F6 and F7 are LOW and F8 is a NIT; none gates.

No finding was relabeled and there are no focus departures to ratify or reject.
