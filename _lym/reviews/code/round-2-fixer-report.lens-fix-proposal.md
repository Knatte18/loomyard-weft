```yaml
lens: fix-proposal
outcome: reported
```

Findings, one per proposal the fixer carried out as proposed:

**F2, fix-proposal lens: no findings.**
- `DroppedCreates` trims and cuts the prefix the same way `isAddedDecisionHeading` does (`internal/discussionparser/identity.go:121-125`), and `addedDecisionSpans` keeps only headings that pass it. The discarded `ok` in `entry, _ := parseAddedDecisionHeading(rest)` therefore cannot be false.
- "Re-run the verb that failed" holds for the record-batch, recover-batch and weaver callers.

**F5, fix-proposal lens: one finding.**
- **LOW.** `namedByAmendment` (`internal/burlerengine/roundcheck.go`) now excludes only the overview. The same defect is open for the other non-card files in the plan directory.
  - `planparser.AmendmentsFileName` is `amendments.md` (`internal/planparser/amendment.go:22`). Its stem `amendments` has 10 characters, which clears `minAmendmentStemLength` (4).
  - `planparser.LensReviewFileName` is `lens-review.md` (line 26), and `IsLensReportFileName` covers the lens reports. Their stems also clear the minimum.
  - An amending entry containing the word "amendments", or the token `lens-review`, exempts those files as a new BLOCKING finding location, though they are no cards.
  - `knownNonCardFiles` (`internal/planparser/validate.go`) already lists the overview, `amendments.md` and `lens-review.md` as plan-level files.
  - The reviewer's proposal and the fixer's `IsPlanOverviewFile` predicate both fix the overview only. An `IsPlanOverviewFile` check on the overview alone leaves the premise "a plan file that is no card is exempt" unresolved for these files.
  - The risk is realistic: amending entries about plan amendments commonly say "amendments".

**F9, fix-proposal lens: no findings.**
- The test hand-writes the round-1 files, so a format change in the engine could break it. It fails loud (`profile.Round == nil` is fatal), so it does not pass silently.

Further calls, one per line:
- Grepped `AmendmentsFileName`, `LensReviewFileName`, `lensReportFileRe` and `isAddedDecisionHeading`, to check what other non-card plan files exist and whether the heading parse in `DroppedCreates` can fail. My context lacked both.
