# Round 1 fixer report

All five findings were fixed in the plan cards; no finding is deferred.
`00-overview.md` is untouched, and `approved:` stays `false`.

## Changes

- F1 (`01-remove-tier1-filing.md`): the `halt_test.go` bullet now drops `LoomSelfreportFiledLock` from `newHaltFixture`'s directory loop and rewords the comment above it.
- F3 (`01-remove-tier1-filing.md`, `04-anomaly-friction-notes.md`): card 1 keeps the `handoffSteps` helper and deletes the filer-asserting handoff test with the other `TierOne` tests; card 4 re-asserts the two-step voucher behavior on the friction directory as one test over the kept helper, and its no-note table keeps only the rows no existing test covers (run lock held, state not `running`, empty history, Tier 2 off, unwritable directory).
  The voucher-match and consume-once rows are dropped, with the existing renamed tests named as their coverage.
- F2 (`06-agent-filed-issues-docs.md`): the stencil bullet now states what the drop rule can let through and what bounds it (trace file named in the note, status and history on disk, `lyx selfreport create` still available), and that the stencil and the `PATTERN-friction-capture` bullet record it.
- F4 (`05-start-vouches.md`): the tests now include a replace-an-earlier-voucher assertion and a no-voucher assertion on the live-strand no-op test, and the card says the consume side is pinned by the renamed existing tests.
- F5 (`03-rename-handoff-voucher.md`): the Intent extends the comment rewording to every Edit target, the closing grep gains `clean-handoff`, and the remaining-match exception names card 4's `anomaly.go` comments.
  The card also now spells out the three renamed test names that cards 4 and 5 cite.

## Deferred

None.
