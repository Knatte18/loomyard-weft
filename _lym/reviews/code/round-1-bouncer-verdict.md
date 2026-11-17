---
verdict: CONTINUE
rationale: "Round 1 carries four gating design findings at MEDIUM (F1, F2, F5, F6), so the decision rule says CONTINUE; no finding was relabeled."
---

The gating findings are F1 (a process-cap or outside SIGKILL of the tier shell reads as a `memory_mb` kill), F2 (a guard kill in `RunAlone` reads as a cap kill), F5 (a `memory_mb` cap that cannot be set fails silently or with the wrong way forward) and F6 (no test pins the tier run's wiring of memory kills).
F3 and F4 are MEDIUM `scope` findings, F7 is a LOW `decision` finding deferred to the parent, and F8 to F10 are LOW; none of these gates.
No review section named focus departures, so there is none to ratify or reject.
No finding cites a PATTERN entry, and no finding's class or severity was changed.
