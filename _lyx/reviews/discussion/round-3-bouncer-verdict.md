---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) sits on the recurring key start-can-seed-during-cancel, open in rounds 1 and 2 and still open, and F3 (MEDIUM, design) sits on the recurring key windows-end-detects-nothing, also open in rounds 1 and 2; both are gating on earlier-open keys, so the loop is circling although the finding count fell from 14 to 5."
---

The decision rule gives `CIRCLING` when a gating finding sits on a key the facts list as open in an earlier round and it is still open.

- F1 maps to `start-can-seed-during-cancel` (rounds 1, 2, 3): the round 2 fix placed the second gate behind batten's pre-run hooks, which seed status lock-free, so the reseed the key names is still reachable. Class `design`, BLOCKING, unchanged.
- F3 maps to `windows-end-detects-nothing` (rounds 1, 2, 3): the Windows survivor refusal still has no producer after the marker-ordering fix. Class `design`, MEDIUM.
- F2 (`tackler-started-unrecorded`), F4 (`weft-leftovers-not-refused-at-start`) and F5 (`nothing-left-refusal-misses-remove`) are new keys in round 3 and do not by themselves trigger `CIRCLING`.

No finding was relabeled, and no Focus departures section is present in the review.
Round 2's other open keys do not recur and are resolved; the review's Rejected section settles the `--remove` re-run, the marker-read ordering and the Windows list ordering.
