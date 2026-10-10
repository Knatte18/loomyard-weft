---
verdict: CIRCLING
rationale: "F1 is BLOCKING design on key d8-reader-write-flags-unbounded, open in rounds 1, 2 and 3 and reopened a fourth time (quoted or escaped option words now bypass the D8 classifier after dash-words and positional words were each closed); F2 is MEDIUM but consistency class, so not gating; no relabels."
---

The D8 read-only classifier has produced a BLOCKING write-bypass finding in every round.
Round 1 found unbounded flags, round 2 bounded dash-words, round 3 found positional words, and round 4 (F1) finds that quoted, escaped, `$VAR` and brace-expanded words slip past the exact-flag check.
Each fix closed the reported instance and left the class open: the classifier is a deny-list of shapes where it needs an allow-list of plain words.
F1 maps to `d8-reader-write-flags-unbounded`, listed open in rounds 1, 2 and 3, and is still open in this round, so the verdict is `CIRCLING`.

Other findings mapped:

- F2 (MEDIUM, consistency) is a new key; it is not gating.
- F4 maps to `d7-refusal-spec-ways-forward`, now LOW decision; F10 to `d7-gate-inherited-strand-name-tests`, LOW design; F3 to `d7-d8-reader-lists-differ`; F7 to `d1-d2-refusals-uncovered`; F11 to `d3-log-in-shared-helper`.
  All are LOW, so none is gating at MEDIUM or worse.
- F9 (LOW design), F5, F6, F8 and F12 are new keys at LOW or NIT.

No finding was relabeled.
The review has no `## Focus departures` section, so there is nothing to ratify or reject.
No entry in this prompt conflicts with another.
