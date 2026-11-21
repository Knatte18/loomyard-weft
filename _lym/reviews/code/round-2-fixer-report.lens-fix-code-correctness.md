```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix for F2, F5 and F9 through the correctness lens, running concrete inputs through each changed branch.

No findings.

**F2.**
- `DroppedCreates` in `internal/discussionparser/append.go` now reads `entry, _ := parseAddedDecisionHeading(rest)`.
- `addedDecisionSpans` keeps only headings that `isAddedDecisionHeading` already parsed, so the parse cannot fail and `entry.Title` is always set.
- `heading`, `rest` and `span.first` are still used, and `fmt` is still used in the file at lines 81, 114 and 196, so nothing is left unused.
- The doc line about the malformed-heading error is gone, and no test asserted that error.
- The `postbatch.go` message now ends "re-run the verb that failed". The earlier grep found the old phrase only in that one source line, so no test or caller depended on it.

**F5.**
- `namedByAmendment` now returns false when `planparser.IsPlanOverviewFile(base)` holds.
- Input: path `_lym/plan/00-overview.md` and an amending text containing "overview".
- Before the fix, the names were `["00-overview", "overview"]`, `containsToken` matched `overview`, and the finding was exempt.
- After the fix, `namedByAmendment` is false, `outsideChanges` is true because `Changed` holds only `c.go`, and the round is refused with "does not list".
- The new `NewBlockingOnThePlanOverviewNamedByAnAmendmentIsRefused` row therefore fails with the defect in place.
- A card file such as `14-card-slug.md` is not the overview, so it still matches as before.

**F9.**
- `writeJudgedRoundOne` writes a complete round 1 with a parsing review, an engine-written fixer-report (`spawned: false`), an acceptance record, a CONTINUE verdict and ledger, and a snapshot with `decisions_routed: false`.
- Round 1 is therefore the highest complete round and is judged, so the producer reviews round 2 and `BuildRoundContext` returns a non-nil context.
- The test asserts `profile.Round != nil`, and the gate passed it, so the round-2 context is built.
- For `Plan-Burler-Review`, `AmendmentTitles` reads `Fix the batch order` and `Round.AmendingTexts` holds that entry's text.
- If the `AmendmentTitles` assignment at `entries_burler.go:186` were dropped, the plan row would get none and the test would fail.
- If it were given to every row, `review-row` would get one and the test would fail.
- Writing the round files from a map has nondeterministic order, but the files are independent, so this is harmless.

No further tool calls made; my context already held every file needed.
