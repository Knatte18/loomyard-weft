---
round: 3
focus:
  - "For every card that deletes, re-signs or changes the observable behaviour of a symbol, grep the repository tests, import allowlists (`scankit.AssertImportAllowlistNoStale` lists) and header comments for readers of the old behaviour, and settle whether each reader is listed as a target of some card; a missing import in an allowlist fails as a stale one does."
  - "Cards 7 and 13 change what cancel and the in-call wait do; settle whether the tier1 and tier2 tests that assert the old status or sleep behaviour (the cancel integration tests, the teardown scenario, the stuck-report and poll-interval floor tests) are Edit targets of those cards."
  - "After the InnerRun short-check cards, settle whether any seam, config key or reason text still describes a poll cadence nothing reads."
  - "For each exported type a card's seam or `Deps` signature names, settle whether some card creates it and whether the consuming card lists it under Uses."
---

The focus lists where the previous round's remaining defects sat, so the closing check goes to the same sites.
