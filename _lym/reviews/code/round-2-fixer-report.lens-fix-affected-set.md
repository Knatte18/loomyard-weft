```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F2:** The only remaining "is not an added-decision heading" message is `internal/discussionparser/append.go:196`. That is a different function, reading headings directly rather than through `addedDecisionSpans`, so the finding does not reach it. No other text says "same webster verb".
- **F5:** `namedByAmendment` is the only matcher in `internal/burlerengine` that exempts a plan-directory file by name, and it now returns false for the overview. The `AmendmentTitles` assignment in `internal/burlercli/validate.go:126` feeds the same `RoundContext` and hence the same `namedByAmendment`, so it is covered.
- **F9:** The only production assignment of `AmendmentTitles` that the finding's premise covers is `internal/shedrecipe/entries_burler.go:186`. The new subtest drives that one.

Calls made:
1. `grep` for "same webster verb" and "is not an added-decision heading": checked the F2 sibling-text set, which my context did not hold.
2. `grep` for overview and `.md` literals in `burlerengine` and `roundcontext.go`: checked for a second overview-exemption site, which my context did not hold.
3. `grep` for `AmendmentTitles` and `amendmentTitlesAt`: checked the F9 wiring sites, which my context did not hold.
