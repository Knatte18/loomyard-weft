```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling cards, rows and cross-references it touches (fix-consistency lens).

**[MEDIUM] Card 7 contradicts itself on which branch the prime's marker commit can reach**
- Changed text: card 7, the F5 bullet ("A one-repository prime on its recorded branch already holds the marker card 5 committed, so the marker commit reaches a prime only off that branch, where it now refuses") against the F14 bound added just above it ("so an operator who sets it to another branch moves the prime's records there").
- Problem: under F14, `lym.primeBranch` can name a branch that lacks `.lym-anchor`. On that branch `refuseOffPrimeBranch` passes, so `migrateWeftMarker` and `recordAdoptedWeftAnchor` commit the marker through `commitAnchorMarker`, which runs `StageAndCommit` and fails mid-merge. Card 10 adds only `takeoverBranch.commitEntry` to the mid-merge route, not these two, so the mid-merge half of F5's premise stays open for them.
- Evidence: `internal/fabricengine/anchormarker.go:79` (`StageAndCommit`). Card 10's Edit list has no `migrateWeftMarker` or `recordAdoptedWeftAnchor`.

**[LOW] Card 7 claims a clone-step order that card 5 doesn't state**
- Changed text: card 7, "`commitAnchorMarker` itself stays as it is, since card 5's clone commits the marker on a fresh prime before `lym.primeBranch` is set."
- Problem: card 5 lists "commits `.lym-anchor` … (`commitAnchorMarker` on the prime)" and "Both one-repository paths set `lym.primeBranch`" as separate bullets with no order between them. The "before" is stated only in card 7.

**[LOW] Card 10's ImpactSummary is stale**
- Changed text: card 10's Edit and Intent additions (`ownMergeInLeftover` own-leftover handling, `takeoverBranch.commitEntry` route).
- Problem: the ImpactSummary still says "Finalize, the pre-merge clear and loom's start refusal gain one way-forward case each". It doesn't mention the abort-and-redo of the row's own unsettled merge-in in `preMerge.clear` and `ownMergeInLeftover`, or the takeover commit's route.

**[LOW] Card 10's inserted sentence takes over the colon that introduces the bullets**
- Changed text: card 10 Intent, "routes … to `commitRecordsAroundMerge`, under the write lock it already holds. `takeoverBranch.commitEntry` takes the same route in that state, under the write lock, which it takes for the route:"
- Problem: the bullets that describe the route for all four committers now hang off the `commitEntry` sentence, and the primitives' sentence ends in a period. A reader sees them as rules for `commitEntry` alone.

**[LOW] Card 10's new two-repository row doesn't match its test's fixture header**
- Changed text: card 10 row `TwoRepositoryParkedMergeContinues` ("a `NewHub` hub").
- Problem: the test's header still reads "`TestRecordsCommit_DuringMerge` (tier1, `NewOneRepoHub`, a parent and a task pair) rows:".

**[LOW] Card 14 doesn't say what a held takeover push does with the committed entry**
- Changed text: card 14 Intent (`takeoverBranch.land` runs the guard) and row `TakeoverEntryPushOfThePrimeIsHeld`.
- Problem: every other push failure in `land` drops the committed entry (`b.dropEntry`, `internal/fabricengine/takeoverwrite.go:190-196`). Card 14 says the caller "returns without pushing" but not whether `land` drops the entry or leaves it committed, nor what `TakeOverRun` reports. The row asserts only that origin is unmoved and that an `operator_unpushed` record exists. Card 14's "a run in the prime waits at its transition" covers transitions, not the takeover.

Tool calls:
1. I grepped cards 6 and 2 for how a task's `--anchor` marker commit carries `AnchorChangeTrailerKey`, to check card 9's rewritten marker rule against them; my context didn't hold card 6 or card 2.
