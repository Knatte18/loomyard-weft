---
verdict: CONTINUE
rationale: "Round 1 carries six BLOCKING findings (F1-F6) and design findings at MEDIUM (F7, F10, F11), so the loop has not converged; no finding was relabelled, and the one focus departure is ratified."
---

The decision rule gives `CONTINUE` on any BLOCKING finding or any design finding at MEDIUM or worse, and the round has both.

F2, F3 and F6 cite PATTERN entries (`PATTERN-test-speed`, `PATTERN-test-only-code`, `PATTERN-code-conventions`).
They stay at BLOCKING with their classes unchanged.
F3 asks for a recorded parent decision, or a declared PATTERN change, before cards 14 and 15 can stand.

## Focus departures

Ratified: the review did not check each importing file of cards 2-11 for whether it drops its `internal/fabricengine` import.
It compared the exported engine names used outside fabric against the Move lists instead,
and the gate's build settles the imports file by file.
