---
verdict: CIRCLING
rationale: "Round 3 carries BLOCKING F1 (design) on the open key d2-glob-bound-assumes-bash-parens-unchecked (rounds 1, 2, 3) and MEDIUM F2 (design) on the open key d6-dead-fork-recover-batch-refuses-at-cap (rounds 1, 2, 3), both still open; F3 (decision) is a new key; no PATTERN entry conflicts and no focus departures were declared"
---

Both gating keys from earlier rounds are open again in round 3.

- F1 (BLOCKING, design) maps to `d2-glob-bound-assumes-bash-parens-unchecked`.
  Round 2 reopened that key on zsh `EXTENDED_GLOB`; round 3 finds a character, `#`, that round 2's own review wrongly cleared.
  Three rounds have each found a further hole in D2's enumerated glob set, so the method is the defect, not one more missing character.
- F2 (MEDIUM, design) maps to `d6-dead-fork-recover-batch-refuses-at-cap`.
  Round 2 asked for a dead-fork-at-cap acceptance row; round 3 finds the dead fork state unreachable, so the case and its rows should go.
- F3 (MEDIUM, decision) is a new key, `d6-exhausted-remedy-uncovered-by-merriam-rung`, first raised this round, and is not gating.

The two LOW round-2 keys were not re-raised: the review holds the default branch and treats the fork-at-cap row as subsumed by F2.
No relabels were made.
No focus departures appeared in the review.
No PATTERN entry is cited or in conflict.
