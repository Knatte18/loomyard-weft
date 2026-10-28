```yaml
lens: fix-affected-set
outcome: reported
```

The fix-affected-set lens checked findings F1 and F2. No finding was left unchecked.

no findings

- F2 (set: the cards that rewrite the "every mode but --require-approved" sentence in `validatePlanCmd`'s Long help): card 7 lists `internal/loomcli#loomCLI.validatePlanCmd` under Edit, and so does card 8. Card 9 now lists it under Edit too, after `plan:internal/loomshed#TestValidatePlan_CardChecks`, and its Extends group no longer has it. No other card in `_lym/plan` names `validatePlanCmd`. Every card in the set is covered.
- F1 (set: the places that state or quote the `test-tier-unstated` way-forward or card rule): outside card 7, the only plan card that mentions `test-tier-unstated` is `_lym/plan/09-doc-go-check.md:44`. That sentence only names the check and deliberately repeats none of its rules, so it is not stale. The other mentions are in `00-overview.md`, which gives a one-line title only, and in earlier `lens-review.*.md` reports, which are records of past runs and are not plan text. The fix covered the spec row, the card rule and the D7 test row inside card 7.

Tool calls:
1. A grep of `_lym/plan/*.md` outside card 7 for the way-forward text and `test-tier-unstated`, plus the group headers of card 8. My context had card 8 only as grep line numbers, so I could not tell which group its `validatePlanCmd` ref was in, and I had not searched the other cards for the F1 text.
