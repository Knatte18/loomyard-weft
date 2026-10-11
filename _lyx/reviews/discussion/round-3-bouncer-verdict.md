---
verdict: CIRCLING
rationale: "F1 (MEDIUM, design, gating) sits on key d7-session-end-no-launch-boundary, open in round 2 and still open: an earlier launch's SessionEnd can still end a resumed launch died, now through the read-alive disjunct added by round 2's fix; F2 (MEDIUM consistency) and F3 (NIT consistency) are non-gating, and no relabel was made."
---

The latest round has no BLOCKING finding, so `CONVERGED` turns on whether F1 sits on a key first raised this round.
It does not.
The review itself says the read-alive disjunct "came in with round 2's F1 fix, which closed the not-yet-registered half of that finding and left this half open".
That is the same defect: an earlier launch's `SessionEnd` ends a resumed launch.
I map F1 to `d7-session-end-no-launch-boundary`, a gating-class (`design`) finding at MEDIUM on a key the facts list as open in round 2, so the rule gives `CIRCLING`.
Severity fell from BLOCKING to MEDIUM and half the gap closed, but the same key still gates.
The reviewer's fix is narrow: drop the read-alive disjunct and add one Death criterion.

- F1: gating, mapped to `d7-session-end-no-launch-boundary`, stays `design` MEDIUM.
- F2: class `consistency`, non-gating; carried as new key `d8-post-clear-reading-guard-unbounded`.
- F3: class `consistency`, NIT, non-gating; carried on `d8-residue-submit-amends-pattern-enter-rule`, which round 2 narrowed to the PATTERN line's wording.
- The review has no `## Focus departures` section, so there is nothing to ratify or reject.
- The review's focus verdicts find D8's confirm reading, the Turn 8 line and the treat-as-empty rejection sound, so those three keys are resolved.
- Every finding cites no PATTERN entry as a violation beyond the amendment bookkeeping for `PATTERN-pane-text-injection`; no relabel and no downgrade was applied.
