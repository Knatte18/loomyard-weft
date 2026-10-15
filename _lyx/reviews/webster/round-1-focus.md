---
round: 1
focus:
  - "internal/creelengine imports: does it stay free of reed and concrete shuttle engine imports, with all delivery going through the Delivery seam, as the overview and PATTERN-creel-store require?"
  - "Sender identity (card 8) and privilege rules (card 7): can a peer claim or forge a sender it does not own, and do privileged verbs check the resolved identity rather than a claimed one?"
  - "Journal fold, retention and daemon restart (cards 6, 9, 10): does the fold reproduce the same pool after replay, and are holds, TTL and reminder state applied consistently on replay?"
  - "Constants-not-config decision: are the named constants (retention, TTL, hold, backoffs, body and reminder caps) defined in creelengine with the stated values and used at every enforcement site, with no config file read?"
  - "Shuttle recordless strand (card 5) and the CLI delivery adapter (card 13): does a send to a strand with no run record stay verified and guarded, and does the adapter gate delivery on the receiver being idle?"
  - "Per-card mechanical checks: did the Rename, Delete and other type-specific checks of each card's groups run against every group's targets, not only the first?"
  - "Test economy: new tests duplicating an existing test's coverage, per-helper tests already covered by a public-surface test, and diff behavior with no test, each naming the covering test or the uncovered behavior."
---

Seed focus drawn from the plan overview's card index and shared decision; no round has been judged yet.
