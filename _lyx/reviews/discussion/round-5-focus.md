---
round: 5
focus:
  - "In `_lyx/discussion/decision-record.md` D6's resume branch (a restarted run that finds a leftover reviewer or fixer half): is there one stated rule that says how each outcome of an attached half (done, died, timed out) is judged, and does it carry over the fresh round's rules, including a fixer done before the marker exists and the review-file byte comparison of D7? Settle whether every liveness combination, including neither half live, is classified."
  - "In D6, what does the round return when stopping a half fails (removal or liveness error), and what stops `BurlerProducer`'s retry from spawning a second pair of halves beside a leftover one? Settle whether the stated bound (no round ends with a live half, no live half spawned twice) holds when removal fails or the half already finished."
  - "In D6, when exactly one half is live on resume, does the Decision state why the live half is stopped and restarted rather than continued, and what bounds the loss of a fixer's partial work?"
  - "In D5 and the Constraints entry for the marker: is the marker's location stated identically in both places, and does the stated bound on releasing the fixer early match what the verb and the lyx writer actually guarantee, including the route of naming another existing path?"
---

Round 4's gating finding sat on D6's resume branch for the fourth round, so the questions above ask for one rule for that branch rather than another case-by-case patch.
