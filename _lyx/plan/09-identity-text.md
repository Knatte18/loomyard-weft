# Card 9 — identity-text

**Create:**
- `internal/discussionparser#IdentityText`
- `internal/discussionparser#TestIdentityText`
- `internal/planparser#IdentityText`
- `internal/planparser#TestIdentityText`

**Edit:**
- `internal/discussionparser/doc.go`
- `internal/planparser/doc.go`

**Intent:** A document segment's identity (D5) is its reviewed artifacts minus what Approve writes and minus what Go and `lyx loom decision add` write into the decision record; each record's sole parser (`PATTERN-sole-parsers`) strips its own part.

`discussionparser.IdentityText` returns the decision record with every `<!-- lyx:carry-over … -->` block, markers included, and every `Added after Discussion` entry under `## Decisions` removed, reusing the scanners `WriteCarryOver` and `ParseAddedDecisions` already use; every other byte stays, so an edit anywhere else changes the text.
`planparser.IdentityText` returns the plan overview with the frontmatter's `approved:` line removed, so `SetApproved` never changes it.
Both are pure functions of bytes; the doc comments say what each removes.

`TestIdentityText` (discussionparser): a record with a carry-over block and one without give equal text; adding an `Added after Discussion` entry through `AppendDecision` leaves it equal; editing a decision's prose changes it.
`TestIdentityText` (planparser): `approved: false` and `approved: true` overviews give equal text; a changed Card Index line changes it.

**ImpactSummary:** Two new pure functions in their sole parsers; no existing caller changes.

**Commit:** 9: parsers: the identity text of the decision record and the plan overview
