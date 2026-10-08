---
verdict: CONTINUE
rationale: "Round 4 carries F1, a BLOCKING design finding (archive-only reset with pending findings has no way forward that makes progress), plus F2, a MEDIUM design finding (no Edit bullet carries the re-sign arrow); no gating finding sits on a key open in an earlier round, so this is not CIRCLING."
---

Round 4 filed 7 findings: F1 BLOCKING design, F2 MEDIUM design, F3 LOW consistency, F4 LOW design, F5 LOW consistency, F6 LOW design, F7 NIT consistency.

F1 and F2 are gating, so the loop continues.

Circling check:

- F1 originates in the round-3 fix for the `refusal-spec-existing-rows-left-stale` finding, but it is a different defect: a reachable state whose named step cannot succeed, not a stale row. It gets its own key, `archive-only-pending-findings-dead-end`.
- F2 is a method-wide gap that no earlier round raised, so it is a new key, `edit-bullets-lack-resign-arrow`.
- The recurring keys in the facts are `refusal-spec-existing-rows-left-stale` and `cards-miss-sites-a-grep-finds`.
  Round 4 does not raise the first; the review's focus verdict 5 settles the changed-text sites.
  The second is mapped to F7, a NIT, so it is not gating.

No relabels and no focus departures: the review carries no `## Focus departures` section, and no finding's class or severity is changed.
