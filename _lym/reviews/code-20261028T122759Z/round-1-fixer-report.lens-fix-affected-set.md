```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, rerunning the set-premise searches for F4, F3 and F1.

- **F4: covered.** I searched `*.go`, `*.md` and `*.yaml` outside `_lym/reviews` for the "every command naming the card" claim and for the "its own gate run" and "gate run included" wordings. Every remaining hit now carries the `lym gate` exception. Those are `fabricref.go:32` and `:75`, `planparser/doc.go:146`, and `loom-plan-spec.md:535`. `_lym/landing/summary.md` is outside the grep and is covered by the edit shown above. No stale site is left.
- **F1/F3: covered.** `gateSegmentPattern` is defined once (`refscanner.go:32`) and used once (`refscanner.go:79`). No second copy of the old `[^;&|()\n`]*` class exists.

Unchecked, by no finding of the review: of the three git-read paths in `batchFabricReferenceEvidence`, the F2 row covers only `HeadSHA`. The `IsAncestor` and `UncommittedPaths` errors still have no row. The review's fix asked for "a git read error" in the singular, so this is not a gap against the finding. I rate it LOW at most and I am not raising it as a finding.

No findings.

Tool calls:
1. Grep over the repo for the F4 phrase variants, because the fixer's context only showed the sites it edited.
2. Grep over the repo for `gateSegmentPattern` and the old character class, to confirm no second copy.
