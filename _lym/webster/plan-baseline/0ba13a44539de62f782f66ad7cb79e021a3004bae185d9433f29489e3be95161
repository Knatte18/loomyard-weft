# Card 2 — decision-ids

**Create:**
- `internal/discussionparser#DecisionIDs`
- `internal/discussionparser#TestDecisionIDs`

**Extends:**
- `internal/discussionparser/doc.go`

**Uses:**
- `internal/discussionparser#CheckDecisionIDs`
- `internal/discussionparser#parseDecisionHeading`

**Intent:** Export the decision record's decision ids, so the parts check (D3) reads them through the record's sole parser and never parses the record itself.

`DecisionIDs` returns the ids `D<n>` of the record's `## Decisions` section, in record order, read the way `CheckDecisionIDs` reads them: each unfenced H3 of the section that `parseDecisionHeading` accepts gives its id, an added-decision entry heading gives none, a malformed heading gives none, and a repeated id is listed once.
A record without a `## Decisions` section gives an empty list.
It shares the section walk with `CheckDecisionIDs` rather than re-deriving it.

`TestDecisionIDs`, a tier0 test in `validate_test.go` beside `TestCheckDecisionIDs`, which already holds the package's decision-id tests, covers rows: a record with several decisions gives their ids in order; an added-decision entry and a malformed heading are skipped; a heading inside a fenced block is skipped; a repeated id is listed once; a record without the section gives none.

The package doc gains one sentence naming `DecisionIDs` as the read of the record's decision ids for other packages.

**ImpactSummary:** One new read-only export over the existing decision-heading walk; no caller changes.
