---
round: 1
focus:
  - "Parent pusher (card 11): do the owner reads, the fresh Bolt pull or run-scope reconcile before them, settle order, drop, hold-back, hand-over and refused-record clearing match D5 and the 'Added after Discussion' entries on every path, including SettleRefused, a dobby tick and a pass interrupted between two steps?"
  - "The round-3 Plan findings: is each of the six built, with its test row, as the cards state, with card text winning where the arbiter entry says the fixer fixed it differently?"
  - "Records hold and teardown (cards 9, 17, 19): enumerate every path that pushes a pair's records, moves or deletes its origin branches, marks its board entry done or removes the pair; does each one honour a standing landing entry and the HeldRecords of a dropped landing?"
  - "Local landing (cards 10, 12, 13, 15, 16): trace a kill between the merge, the landing record, the push-now mark and the records commit; does a resume record the owed tip, read the run as past its merge and never land the work twice? Does every local landing leave the trailer commit?"
  - "Pusher and ledger concurrency (cards 4 to 8): can a spawn, push-now mark or newly owed commit be lost when it races a pusher that is exiting, removing the entry or holding its lock? Are ledger read-modify-write cycles and lock ordering safe, and does the board target commit uncommitted writes before it pushes?"
  - "Reset guard (card 18): does every path that can force or move an origin branch past a tip go through the D7 reachability rule, and does a failed or partial reflog or fetch read refuse rather than reset?"
  - "Network wait (cards 2, 3, 20): do both drivers, lym's and batten's, re-step only git-transport and github-api, keep and clear the wait record and driver lock correctly on every exit, including cancel and a class change, and do status, run watch and summary read them consistently?"
  - "Merge-in leftovers (card 14): can the removal delete anything outside D9's bound (tracked, not ignored by the base's rules, symlink, nested repository, path with special characters), and is the base file written, reused and removed correctly across first, later, fast-forward and resumed merge-ins?"
  - "Loosened checks: compare every removed or widened assertion in tests that exist at the base against the Loosened-Check trailers, including ones no trailer names, and confirm each trailer states a measured cause and that a claimed replacement test holds the behaviour no looser."
  - "Docs and wording criteria (cards 22, 23): do both edit directives, the doc comments and reflection template D6 names, and the PATTERN-pair-remote line and backgrounds match the declared text and the behaviour the code now has, with no stale description of a synchronous landing push or the old pusher backoff left behind?"
---

Seed focus for round 1, drawn from the decision record, its added entries and the plan overview.
The diff has not been reviewed.
