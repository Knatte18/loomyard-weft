---
round: 1
focus:
  - "Fidelity: read `_lyx/discussion/decision-record.md` and settle whether each Decision and Constraint is carried by some card (cards 3, 4 and 5 name Decisions 1-6), and whether any card adds scope the record does not license, for example the `StatusPushLockWait` value, the `LockWaitUnbounded` constant or the `PATTERN-push-both-sides` wording."
  - "Granularity in card 3: it creates five symbols and edits fabricengine, battencli and loomcli files. Settle whether the two constants, the sentinel and `IsPushRejected` are each independently reviewable or belong with `PushPairAnchored`, and whether the battencli and loomcli call-site edits belong in this card."
  - "Granularity and sequencing across cards 3 and 4: both edit `internal/loomcli/wiring.go` (card 3 passes `StatusPushLockWait`, card 4 then moves `Push` to `PushPairAnchored`). Settle whether the split is a sound unit boundary or leaves card 3's wiring edit and card 4's rewrite of the same line in conflict."
  - "Test economy: settle whether the table-driven tests planned in cards 1 and 2 and the lock test in card 3 duplicate behavior an existing test already covers (`TestTryAcquireWriteLock`, the `HasUnpushed` tests, the existing `CoalescePushBothAt` lock tests), and whether the card 4 and card 5 integration steps repeat the card 3 scenario test's rejection and divergence paths instead of covering behavior only they own. Name the covering test for any finding."
  - "Attack-surface in `**Intent:**`: settle whether each card that adds a retry, a bounded lock expiry, a new verb behavior or a changed guard states what it lets through and what bounds it. Check card 1 (new bounded acquisition), card 2 (the no-upstream fold into `true`), card 4 (lock expiry and every push error warn and continue) and card 5 (unconditional both-sides push in the prime) against that."
  - "ImpactSummary conclusions: settle whether the `Edit` cards' summaries (cards 1-5 where present) state a blast-radius conclusion rather than restating `Intent`. Card 3's summary lists contenders, and card 4's lists changed surfaces; check whether a caller count or a scope bound is stated."
  - "Verify coverage: settle whether every package a card targets (`internal/lock`, `internal/gitrepo`, `internal/fabricengine`, `internal/battencli`, `internal/loomcli`, `internal/statuscommit`, `internal/fabriccli`, and the `PATTERN.md` and `docs/overview.md` edits) has its tests run by the `## verify:` section, hermetic tagged tests included."
---

Seed focus only: it names where to look and which question to settle, and sets no verdict.
`Custom` is not used by any card, so no focus entry covers its typing rules.
