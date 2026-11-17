```yaml
lens: fix-proposal
outcome: reported
```

I challenged the four proposals the fixer carried out as written: F2, F3, F4 and F5. F1 was fixed with the fixer's own change, so it is outside this lens.

1. **LOW** — Card 8, the `Hold` seam (F4 as proposed): "the first hold standing in the local Bolt, whichever machine wrote it".
   - Summary: "first" can pick the wrong hold when this machine holds several, and step 1 then refuses a `first` its other hold covers.
   - Evidence: `internal/boardengine/holdwrite.go:24` says "A hold of the same machine with another kind or other subjects is appended beside the standing one", and `Owners.Holds` is a list (`owners.go:56`).
   - Concrete case: one adopt hold over {a} and one adopt-fix hold over {b}, both this machine's. `lym adopt check b` reads the {a} hold first and refuses "a `first` outside the hold's subjects".
   - The seam should return the hold of this machine that covers `first`, or else the foreign hold that stands. Only one foreign hold can stand, because `WriteHold` writes nothing over another machine's hold (`holdwrite.go:22`).

2. **LOW** — Card 3, step 5 (F2 as proposed): "so a later push from this machine never publishes a takeover entry for an adopt that did not finish".
   - Summary: the guarantee covers only the parent's records branch. Step 4 pushes the records side first, so when the parent's rebase conflicts, the moved pair's records branch may already hold the entry on origin.
   - Evidence: step 4 reads "commits each and pushes it, records side first", and step 5 reads "a records branch whose push landed is left as it is". Card 3 also says "the takeover entry, once pushed, stays".
   - The sentence should say "on this branch" so it does not claim the whole adopt publishes no entry.

I found no defect in F3's new tier 1 rows. `subjectOwnedElsewhere` (`check.go:426`) reads Bolt owners, which no card deletes; card 9's `ownedElsewhere` is a different function (`adopt.go:411`). runCheck's owned-elsewhere path returns before any gate runs (`check.go:158-164`), so the row "no gate runs" holds. I found nothing wrong with F5 either: `greenFixture`'s `dir` and `otherDir` are read only by the count-reset assertion (`green_test.go:150-161`).

Tool calls:
- grep of boardengine for the `Hold` and `Holds` types and of card 9 for owner reads, plus check.go lines 150-165: to learn how many holds can stand and whether runCheck's owner path survives.
- registry.go `Hold` struct and card 9's delete list: to see whether card 9 deletes `ownedElsewhere`, the function the new row depends on.
- check.go `hubCheckDeps`'s `OwnedElsewhere` fill and adopt.go `ownedElsewhere`: to tell the two functions apart.
- `subjectOwnedElsewhere` body and its plan mentions: to confirm it reads Bolt owners and no card deletes it.
- holdwrite.go: to check whether one machine's holds can stand side by side.
- grep of card 2 for multi-hold rules: to see whether card 2 limits a machine to one hold. It does not settle that.
