---
verdict: CONTINUE
rationale: "Round 1 carries a BLOCKING design finding (F1, plan-gate-only bound false on the shared planparser and resolvePass seams) and MEDIUM design findings F2, F4 and F5; no departures or relabels."
---

The first round has no prior ledger, so every finding is new.

Gating findings (class `design`, MEDIUM or worse):

- F1, BLOCKING: D3 and D5 claim the new checks never run at dispatch, but `planglyph.ValidateDispatch` calls `planparser.ValidateFormat` and `resolvePass` is shared.
- F2, MEDIUM: D1's first-card test depends on checks D7 builds later.
- F4, MEDIUM: D5's reference scan is not the existing scan and its limits are unstated.
- F5, MEDIUM: the re-sign arrow's head has no stated bound or consumer.

F3 (MEDIUM, `decision`) is not gating by class, but the fixer still fixes it.
F6 to F9 are LOW or NIT and non-gating.

The review has no `## Focus departures` section, so there is nothing to ratify or reject.
No finding is relabeled in class or severity.
No finding cites a PATTERN entry.
