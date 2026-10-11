---
round: 2
focus:
  - "Journal write path: check whether any card creates the directory that holds the renumber journal before the first temporary write, and whether any planned test uses a journal path whose parent does not yet exist."
  - "Attack-surface bounds in the loom card-verbs card: check that the Intent states, for each of the three verbs separately, what it writes or removes and what bounds it, including what restore-renumber discards."
  - "Test economy across the planparser renumber, planparser restore and loom card-verbs cards: check that no verb-level test re-asserts a behavior a planparser test already pins, and that success cases are not one test per verb."
  - "Lease test determinism: check that the planned lease test cannot pass when the verb released the lease early, independent of goroutine scheduling."
---

Focus entries target the open `design` findings of round 1; the lower-severity findings remain in the ledger for the fixer.
