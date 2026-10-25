---
verdict: CONTINUE
rationale: "Round 1 carries three BLOCKING design findings (F1, F2, F3) and eight MEDIUM design findings (F4 to F11), all gating; no relabels, and no key recurs from an earlier round."
---

The latest round is the first, so no fix has been reviewed and nothing converges yet.
F1 (`runproc.End` kills the passes it should leave), F2 (a prime anchor change strands other machines' primes) and F3 (the recovery cannot tell its states apart) are BLOCKING and design-class.
F4 to F11 are MEDIUM design findings.
F12, F15 and F16 are consistency; F13, F14 are LOW design.

No finding is relabelled in class or severity.
No `## Focus departures` section appears in the review.
No declared-change entry is cited as violated: the review cites `PATTERN-refusal-way-forward` only through F2's way-forward argument, and that entry is judged by the overview's current text.
