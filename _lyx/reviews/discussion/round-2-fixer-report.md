# Round 2 fixer report — discussion

Every finding was checked against the code and holds; all seven are fixed.
`lyx loom validate-discussion` passes after the edits.

## Fixed

- **F1 (MEDIUM, design): named catch reads no carry-over.**
  Confirmed: `loom-rubric-plan-review.md` holds the record as "the measuring stick and never the subject", and `landing-template-describe.md` forbids naming review rounds.
  D2 now names each consumer and what its stencil tells it: the Plan-Write and rework stencils check each listed finding against the code and never plan it as scope, and the Describe stencil carries each Plan-Review or Webster-Review finding into the items to check by hand, worded without review rounds.
  The Plan-Review and Webster-Review rubrics are stated as unchanged.
  Scope In names the three stencils.
  The Open risks bullet names those guards and states that a missed wrong Discussion-Review fix becomes part of the Plan-Review answer key.
  The support log's Rejected alternatives records why the rubrics do not check carry-over entries.
  An acceptance criterion covers the stencil changes.
- **F2 (MEDIUM, design): degraded ledger read empties the earlier-open set.**
  Confirmed: `ledgerHistories` skips an unreadable or unparseable ledger.
  D1 now renders a parse-error line per missing or unparseable earlier ledger, and the lighter rule says such a line means round 1's rule.
  The D1 Bound lists that case, and an acceptance criterion covers it.
- **F3 (MEDIUM, decision): entry identity without a format.**
  Confirmed: `BouncerConfig.Name` is documented as a log and error identity only.
  D2 now fixes the entry's shape as opening and closing HTML-comment marker lines carrying the segment name, says the entry is Go-owned, so a fixer's edit between the markers is overwritten, and names a new told config field for the segment name.
  Acceptance criteria cover the markers and refuse a missing closing marker.
- **F4 (MEDIUM, design): Validate cannot catch the failure that matters.**
  Confirmed: `Validate` checks file existence and heading lines only, and the Bouncer is told only `DecisionRecordPath`.
  The D2 Bound drops `Validate` and replaces it with a check that the bytes outside the entry are unchanged.
  Judge-written fields are rendered on one line with backticks escaped, and a failed check leaves the record unchanged and returns an error.
  The support log records the rejected `Validate` bound.
  An acceptance criterion covers a key that contains a line break, a backtick or a marker.
- **F5 (LOW, consistency): docs list omits discussionparser.**
  Confirmed in `internal/discussionparser/doc.go`.
  `discussionparser` is added to the Constraints' docs list, with the reason.
- **F6 (LOW, decision): path form, rounds source and unlabelled entries.**
  D2 now says paths are anchor-relative and rounds come from the ledgers' own round numbers, as `ledgerHistories` takes them.
  Unlabelled open entries are listed as `unlabelled`.
  The Auto-mode assumption and the acceptance criterion now match.
- **F7 (LOW, decision): stale-entry lifetime.**
  D2 now says a skipped review (`approveWithoutReview`) leaves the entry as it is.
  An entry stays until its segment's next judged `Done`, and its paths may point into an archived run directory until that `Done` arrives.
  An acceptance criterion covers the skipped review.

## Disputed

None.

## Deferred

None.
