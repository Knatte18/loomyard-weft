```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked each claim the fix added or rewrote against the code at HEAD.

No findings.

These claims hold:
- **Card 5 slot ask:** the refresh "runs the map run's measurement in the map run's slot ask". `testMapRun.holdSlot` acquires `mapSlotAsk(r.tiers)` through `AcquireSlots` (`internal/gatecli/testmappackage.go:490-492`).
- **Card 13 machinery:** it mirrors `redundantCmd`'s `(goBinary, minuteUnit, grace)` signature (`internal/gatecli/redundant.go:105`) and reuses `mutationCheck.holdSlot` (`internal/gatecli/redundantmutants.go:528`).
- **Card 7 match:** it uses `goFileHashes` (`internal/gatecli/redundancytests.go:215`) and `Fabric.ChangedPathsBothSides` (`internal/fabricengine/parenttips.go:129`). Both exist.
- **Card 2 absent directory:** "as an absent cache does under `PATTERN-lym-files`" matches `pattern/PATTERN-lym-files.md:26`, "An absent cache reads as empty and refuses nothing."
- **Card 4 and card 5 doc targets:** the gateslot doc's bounds paragraph (`internal/gateslot/doc.go:174-189`), its "holds the larger of tier0's and tier1's slot asks" sentence (line 185) and its refusal list naming the report (line 169) all exist as the cards name them.
- **Card 11 merge base:** `parentMergeBase` (`internal/gatecli/scope.go:692`) returns `(string, bool, error)`. That fits the `mergeBase` closure's no-parent-branch answer.

Tool calls:
- One grep for `holdSlot`, `redundantCmd`, `goFileHashes`, `parentMergeBase`, `AcquireSlots`, `ChangedPathsBothSides` and the absent-cache rule in `PATTERN-lym-files`. My context held none of these definitions.
