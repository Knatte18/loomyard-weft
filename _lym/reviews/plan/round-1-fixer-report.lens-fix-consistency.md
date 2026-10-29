```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: checking that the round's changes agree with the sibling cards and cross-references they touch.

**[LOW] Card 13's wait Bound sentence is stale after the F4 change**
- **Changed text:** `_lym/plan/13-pr-review-row-waits.md`, the closed-and-not-merged verdict.
  It now says that "with `Continue` on a dirty tree or an ungated head it keeps waiting".
- **Problem:** The unchanged Bound line just below still says the wait "lasts until the pull request is approved, merged or closed, the operator continues, or the run is cancelled".
  A closed pull request no longer always ends the wait.
  A reader of the Bound misses this one case where the wait goes on.
- **Evidence:** In the same card, the closed verdict keeps waiting on `Continue` with a dirty or ungated tree, while the Bound line lists "closed" as an ending.

Twins I checked that agree:
- The seam-test targets moved from card 4 to card 11's Edit list, and no other card lists them under Uses.
- Card 14 still edits `TestLandingDeps_EveryFieldPopulated`, which it needs for its `unfilled` map.
- Card 11's step-8 `UpdateRecord` matches card 13's claim about it.
- Card 15's "only PR-Review records `ClosedForContinue`" matches card 13.
- Card 8's new halt rows (`https://@github.com/o/r`, `ssh://git@github.com:22/o/r.git`, `git://github.com/o/r`) agree with card 1, which still refuses those shapes.
- Card 5 now extends `internal/shuttleengine/doc.go`, while card 6 keeps it under Uses; card 5 lands first, so that order is fine.
- Card 11's change to the `loomrecipe/doc.go` sequence matches `wantPawlRows`.

Calls I made:
1. Read cards 1, 12, 9, 17 and the overview, to check twins of the resolver rows and the continue flag. My context did not hold those cards.
2. Searched the plan for every target the seam-test fix touches, to rule out a uses-later-target. I also searched the docs for the pawl sequence, to find stale twins.
