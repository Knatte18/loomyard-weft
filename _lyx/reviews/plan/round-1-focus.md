---
round: 1
focus:
  - "Fidelity: check every Decision D1-D7 and every Constraint in the decision record against the cards. Name any D-item no card carries, especially D3's pre-clear box reading, D4's post-`ErrSubmissionNotLanded` signal re-read, D6's restored-boundary tick and unsent-re-prompt mark, and D7's `allOutputFilesExist` guard."
  - "Card order: cards 5, 6 and 4 consume sentinels and helpers from cards 2-4, and cards 8-10 consume card 7's `Run.Stop` and `Runner.StopStrand`. Does any card use a symbol a later card creates? Does card 10 delete a remover before cards 8 and 9 drop their last caller?"
  - "Re-sign arrow and member glyphs: for each `Edit` card whose `Intent` changes a signature (for example `sendVerified`, `confirmSubmitted`, `handleGatedBoundary`, `burlerengine.Handle`), does the target carry the arrow, and do Go-file-only targets list member glyphs?"
  - "Test economy: do the new tests in cards 3, 4, 5, 6 and 7 (`stop_test.go` and its tables) duplicate behavior an existing test in `attach_test.go`, the send tests or the wait-loop tests already covers? Name the covering test."
  - "Attack-surface question: card 3 (the 10 s idle-pane override of an unmatched `turn_start`), card 4 (extra Enters, the clear choreography keys) and card 7 (a verb that kills a strand and writes a terminal record). Does each `Intent` state what the change can skip or let through and what bounds it?"
  - "`verify:` coverage: does the section run the tests of every package any card targets (`shuttleengine`, `claudeengine`, `burlerengine`, `websterengine`, `testkit/shedfake`, `shedengine`, config and PATTERN-enforcement tests), in the template's order?"
  - "Granularity: is card 7 (a new verb, a record lock and an attach change in one card) one independently reviewable unit, and are cards 8 and 9 split along a seam a reviewer would use?"
---

Focus is a starting point; every rubric item still applies.
