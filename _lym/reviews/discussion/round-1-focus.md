---
round: 1
focus:
  - "Decision 1 removes the `runDobby` pre-check and moves `registerDobby` behind the lock. Does it state what a process that loses the lock can still reach or skip, and is the claim that every lock prober holds only between a try and a release backed by the code the writer read?"
  - "Decision 2 (re-exec lock wait) and Decision 3 (refused-re-exec exit): is the claim that a second dobby spawned inside the exec gap is the only way to lose the lock bounded, and does Acceptance criteria cover the bound it states?"
  - "Decision 8 rests on a claim about the state record: that a machine's record still lists landings made but not yet tidied after ownership moved away. Is that fact recorded with its evidence, or left for Plan to find out?"
  - "Decision 9: the reconcile migration copies a tuned `finalize` value into `adopt`. State what it can now let through (a hub's `finalize` tiers now govern the adopt check) and what bounds it; check the Open risks line against that."
  - "Decision 10: the hidden `lym adopt check` verb runs the hub's gate and posts to the orch. Check that the bound covers what it can skip or let through, including a second concurrent check in the same prime and a stale `check.json` pid."
  - "Decision 5 and Decision 11 against Acceptance criteria: does every stated behavior and bound (`dobby.out` growth between spawns, a dead-pid check record) have a matching criterion, and does any criterion test something no Decision states?"
  - "Relocation pass: apply the completeness-before-leanness test to Rationale paragraphs, the Open risks lines and the Constraints bullets that restate a Decision. Name anything that duplicates a Decision or belongs only in `support-log.md`, and extract any requirement the plan writer needs first."
---

Seed focus for round 1.
No amendment is under review.
No Decision edits the task's own board entry or files a task, so those bullets are not listed.
