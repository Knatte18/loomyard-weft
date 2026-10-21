---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) is the same enumeration-gap method failure as key new-keys-break-exact-set-pins-and-docs, open in rounds 1 and 2 and still open in round 3, now at a reflection-walking NilSeams audit; F2 to F5 are LOW scope and first raised in round 3."
---

F1 is the only gating finding.
Its own text calls itself "the same method failing on a site those fixes did not name", so I map it to `new-keys-break-exact-set-pins-and-docs`, which the facts list as open in rounds 1 and 2.
That key is BLOCKING design in round 3, so the decision rule gives `CIRCLING`.

No finding is relabelled and no focus departures section exists in the review.
F2, F3, F4 and F5 are LOW scope on new keys and do not gate.
The round-2 key `card2-generation-start-skip-no-bound` is resolved: the review's Rejected section says card 2's `GenerationStart` mark answers the attack-surface question.
