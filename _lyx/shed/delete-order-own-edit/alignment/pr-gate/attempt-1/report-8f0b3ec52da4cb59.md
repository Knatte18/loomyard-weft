---
verdict: APPROVED
---

## Measure

The board entry `delete-order-own-edit` as it stands; the decision record has no `Added after Discussion` entries, so nothing amends it.

## Diff against the entry

PR 622 at `b896a56ca` against `main` changes only `internal/planglyph`.

- Fix point 1, the exemption in `deleteOrderFindings`: `unrewrittenLaterEditRegions` in `deleteorder.go` skips a reference line that an Edit region of a card numbered at or below the deleting card holds, and both callers pass through it, so the plan-gate path in `callercoverage.go` and the scan-only `LaterDeleteReferences` agree.
  `LaterDeleteReferences` builds the deleting cards' regions as well, only for the exemption, so the begin-batch case in the entry's Seen section, where card 18's own Edit covers `TestPlanTable`, is no longer reported.
- Fix point 2, the test: new rows in `TestLaterDeleteReferences` pin the deleting card's own Edit and an earlier card's Edit as exempt, and a reference that only a later card covers as still blocking.
  The changed row in `glyphchain_test.go` is the plan-gate path's old expectation that the entry reverses.
- Fix point 3, the docs: `doc.go`'s delete-before-reference entry states the exemption.

No change goes beyond the entry.
`callercoverage.go` and `glyphchain_test.go` are not in the record's scope list, but they are the plan-gate caller of `deleteOrderFindings` the entry names and its test, which is not a scope deviation.
The quoted-import arm left out of scope is not named by the entry.

## Open risks

The record carries one open risk: a deleting card's Edit target with an unreadable answer now raises glyph-rejected at begin-batch.
It names no parent correction, so it does not escalate.

## Overlap

- `overview-seats` (row 12, blocked at Webster): no shared path; its changed paths hold nothing under `internal/planglyph`.
  This run unblocks it, as the entry's After-it-lands section says, which is no design conflict.
- `seed-vscode-config` (row 0) and `twill-fork0` (row 7): no changed paths, no overlap.
- Parent commits: none listed.

No overlap, so no landing order applies.

## Superseded

None.
