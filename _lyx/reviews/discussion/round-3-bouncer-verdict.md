---
verdict: CIRCLING
rationale: "F1 (MEDIUM, design) is a third round of overclaim in D2's Bound, now a resume path that finalize's strand removal rules out, so I map it to the open key d2-bound-overstates-probe-reach (rounds 1, 2); F2 is a new LOW consistency key and no relabel was made."
---

Round 3 carries two findings and no BLOCKING one; the highest severity fell from BLOCKING in round 2 to MEDIUM.

F1 is gating-class (`design`) at MEDIUM.
It targets D2's Bound, where each round's fix has added a claim about what the probe or a resume reaches that the code does not support.
Round 1 and round 2 raised the same overreach in the same Bound, so F1 maps to `d2-bound-overstates-probe-reach`, which the facts list as open in earlier rounds.
It is still open, so the decision rule gives `CIRCLING`.

F2 is a LOW `consistency` finding on D2's Rationale ("finalize stays the one place an expired request is marked").
It is a new key, `d2-rationale-finalize-sole-expiry-claim`, and does not gate.

The review notes no finding on D11, so `d11-proof-does-not-isolate-hop` resolves.
The three round-2 consistency keys (`scope-only-judging-decided-and-unconfirmed`, `ledger-interval-location-contradicts-d1`, `d1-interval-precedent-and-never-pinged`) are not raised again and resolve.

No finding was relabeled.
There is no `## Focus departures` section in the review, so nothing to ratify or reject.
