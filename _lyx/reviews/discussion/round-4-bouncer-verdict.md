---
verdict: CONTINUE
rationale: "Round 4 carries one gating design finding at MEDIUM (F2, stop-on-failure assumes serial steps while Decision 4 requires t.Parallel per subtest), and it is a new facet rather than an unchanged gating key, so CONTINUE and not CIRCLING."
---

The latest round has no BLOCKING finding, but F2 is class `design` at MEDIUM, which gates.

F1 and F3 are MEDIUM but class `decision`, so they do not gate on their own.
I did not relabel any finding, and the review carries no `## Focus departures` section to ratify or reject.

F1 and F3 sit on the same scenario-merge decision as the round-3 keys `scenario-steps-isolation-bound` and `scenario-merge-per-family-measurement`, which were design MEDIUM then.
Round 3's text added bounds, and round 4 finds the next holes in them: the or-clause, missing margin and cache handling.
That is progress with a narrowing class, not a repeat of an unchanged finding.
F2 raises a question round 3 never asked (parallel steps), so it is mapped to a new key.

No PATTERN entry is cited as violated by any finding.
`PATTERN-test-speed` is relevant to the fix F2 proposes, and the entry permits a comment-named global-state exemption from `t.Parallel`.
