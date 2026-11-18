---
round: 3
focus:
  - "Cards 3 and 9 add required seams (Drift.ChangedFiles, Drift.CommitMessages, PullRequestDeps.ShedStatus) that batten's wiring leaves nil: check internal/battencli/wire_seams_test.go intentionallyNil and TestWire_EverySeamFilled against every seam any card makes required or fills, and settle whether each card lists every table it must edit."
  - "Card 7 changes the signature of internal/shedrecipe captureReview or a sibling helper: check every test in internal/shedrecipe that stores or calls that helper, including table fields of a fixed func type in entries_burlerhalves_test.go, and settle whether the card lists each one under Edit and carries the re-sign arrow where parameters change."
  - "For every card that changes a function signature, a required seam or a routing value, settle whether each test, test kit or table that pins the old shape is listed under Edit."
---

Round 2 found two unlisted-test gaps; the entries ask the reviewer to sweep for the same defect shape across the remaining cards.
