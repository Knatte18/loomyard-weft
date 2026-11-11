```yaml
lens: fix-affected-set
outcome: reported
```

no findings

My list: all five findings were checked, and none was left unchecked.

- **F1:** I searched the repo for every use of the outcome labels (`outcomeNothing`, `outcomeSwept`, `outcomePushed`, `nothing to sweep`). The only consumers are `internal/machinecli/set.go` and its test, so the new `outcomeKept` is covered. No doc or spec lists the outcome values, so none went stale. The `pushed` upgrade at `set.go:264` fires only for `outcomeNothing`, which a lost-only hub no longer has.
- **F2 and F3:** these two cover test gaps, so they have no set of callers to search. Each is a single new test row.
- **F4:** the only non-test caller that sets `Replace` is `set.go:197`, and it passes `Replace` alone. The new guard rows cover Slugs, Prime, Claim and Takeover, the four fields the guard checks.
- **F5:** the doc comment sits on `OwnerRequest`, which has a single declaration.

Tool calls made:
1. Searched the repo for the outcome labels, to find every consumer of the F1 constants.
2. Searched the Go files for `Replace:` and `Replace` beside Slugs or Takeover, to find every caller that sets the replace set.
