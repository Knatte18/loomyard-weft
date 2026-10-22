---
round: 2
focus:
  - "Card 3 (`_lyx/plan/03-verifytree-pause-hold.md`): does `PauseHold` expose every production interval a `TestVerify_PauseHold` row would wait out (Verify's PastGrace poll, WaitUncovered's sleeps), per PATTERN-test-speed's injectable-time rule?"
  - "Card 3: after `Hold.Wait` returns having waited, does `Verify` redo its dirty-tree and skip checks before it writes the running marker, so no site verifies a dirty tree (PATTERN-verified-tree)?"
  - "Card 3: does the `Waiter` Notify wiring call `held` only when a hold actually starts, not for `WaitUncovered`'s unreadable-record notice that returns nil?"
  - "Cards 8 and 9: does the decision record carry an `Added after Discussion` entry for the PATTERN.md size departures (entry-line length limit, overview headroom) that these cards rely on?"
  - "Card 14 (`_lyx/plan/14-tender-resume-claims-paused.md`): when the resend after an uncovered claim is refused again with a PausedError, is the session claimed into the covering entry, as the decision record's rule for a send refused by another live entry requires?"
  - "Cards 2, 9, 10 and 15: do their Intents state the failed-group-signal bound, SendConditioned's coverage read, the dropped refused segment-color line, and creelengine `doc.go`'s opening seam sentence, without adding targets that `redundant-package-target` or `redundant-file-target` refuse?"
---

Round 1 left two BLOCKING design findings on card 3 and two MEDIUM decision findings on cards 8, 9 and 14.
The entries above ask the reviewer to settle each question on the plan as it now stands.
