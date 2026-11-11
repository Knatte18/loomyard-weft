---
verdict: CONTINUE
rationale: "Round 1 carries four gating design findings at MEDIUM (F1, F3, F4, F5), so the loop has not converged; F2 (decision), F6, F7 (design LOW) and F8 (consistency) are non-gating; no finding relabelled."
---

Gating findings at MEDIUM or worse: F1 (own test file embeds invisible to the inert carve-out), F3 (method bodies never exercised), F4 (listed D5 cases without a row), F5 (go:embed pattern shapes unpinned).
F2 is class `decision` and goes to the parent; F6 and F7 are design at LOW; F8 is consistency at LOW.
The review has no `## Focus departures` section.
No finding is relabelled.
