---
verdict: CONTINUE
rationale: "Round 1 carries three BLOCKING design findings (F1, F2, F3) and three MEDIUM design findings (F4, F6, F7), all gating; no relabel was applied."
---

Round 1 is the first round, so no fix has been reviewed yet and nothing can converge.

- BLOCKING design: F1 (hub-wide pause has no bound on its caller), F2 (parent-lock exception lets a covered run land during its pause), F3 (a covered lock holder still freezes other runs).
- MEDIUM design: F4 (orch interrupt skips the grace and the report), F6 (no way to move a loom run out of the parent lock queue), F7 (post-kill stop has no bound).
- Non-gating: F5 (MEDIUM consistency), F8, F9, F10, F11 (LOW consistency).

No finding cites a PATTERN entry directly, and no departure was made from any finding's class or severity.
The review has no `## Focus departures` section.
