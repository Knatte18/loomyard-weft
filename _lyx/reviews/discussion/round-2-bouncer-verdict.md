---
verdict: CONTINUE
rationale: "Round 2 has no BLOCKING finding, but F1 (MEDIUM design, D6 amendment-as-retry rule) maps to ledger key d6-dead-fork-recover-batch-refuses-at-cap and F2 (MEDIUM design, zsh EXTENDED_GLOB) maps to d2-glob-bound-assumes-bash-parens-unchecked, both keys open in round 1, so the rule gives CONTINUE."
---

The latest review carries two gating-class findings at MEDIUM.

- F1 is the D6 dead-recovery rule: the same decision and the same recover-batch route that round 1's BLOCKING key covered, now with a different gap (an unrendered amendment below the cap, and the disagreement with Merriam's ladder).
  I map it to `d6-dead-fork-recover-batch-refuses-at-cap`, reopened.
- F2 is the D2 shell-dialect bound, the same weakness as round 1's glob-bound key, now on zsh `EXTENDED_GLOB` and a leading `^`.
  I map it to `d2-glob-bound-assumes-bash-parens-unchecked`, reopened.

F3 (LOW scope) and F4 (LOW consistency) are non-gating and get new keys.
No finding was relabelled and the review carries no `## Focus departures` section.
No PATTERN entry is cited by any finding.

The other round-1 keys are resolved: the review's "no finding" verdicts clear D3, D5, the quoted-glob rule and the `ls-remote` form, and the remaining non-gating keys were not raised again.
