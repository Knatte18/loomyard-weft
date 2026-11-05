---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) sits on key relay-restart-cap-unreachable, open in rounds 1 and 2 and still open in round 3, so the decision rule gives CIRCLING; no finding was relabeled and no departure was named."
---

F1 is the only gating finding.
It maps to `relay-restart-cap-unreachable`, which the facts list as open in rounds 1 and 2.
Round 2's fix moved the `StartFailures` reset to the relay's first accepted read, and round 3 shows that reset lets a relay that dies after its first read respawn once a minute with no cap.
The next fix has to judge a relay by its lifetime or otherwise cap the spawn count across a post-first-read death.

F2 to F7 are on keys first raised in round 3 and are not gating: F2 and F3 and F5 are `scope`, F4 and F6 and F7 are `consistency`.
F2 is MEDIUM but not `design`, so it does not block.
The round 2 keys `creel-doc-extends-falsified`, `paused-error-after-post-unspecified` and `newroot-long-missing-slack` did not recur: F6 concerns the `ReadWait` contract wording, F2 a different error site, and the round 3 review rejects the other two.
There were no `## Focus departures` in the review.
