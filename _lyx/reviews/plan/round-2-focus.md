---
round: 2
focus:
  - "Card 5 (`_lyx/plan/05-hub-store-gc.md`), the gc-failure subtest and the Intent's description of gc failures: settle whether the failure the test drives makes `git gc --auto` exit non-zero, given that `git gc --auto` exits 0 silently when a live process holds `gc.pid`."
  - "Card 2 (`_lyx/plan/02-gitrepo-retrying-read.md`), the `parity_test.go` regression table for the stale-pack-index bug: settle whether each row reaches a stale index on its own, given that one `Repo`'s first object-not-found reindexes and updates `lastPackFingerprint` for later reads."
  - "Cards 1 and 4, test fixtures that depend on go-git walk order or process reaping: settle whether each planned assertion is deterministic as specified."
---

Focus targets the design-class questions from the round-1 judging; other findings stay for the reviewer's normal pass.
