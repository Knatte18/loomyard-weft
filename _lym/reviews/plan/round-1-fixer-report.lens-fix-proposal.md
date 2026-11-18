```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out, through the fix-proposal lens.

**Findings**

1. **MEDIUM. Card 14's new `takeoverBranch.land` guard (the F6 proposal) never says what happens to the takeover entry.**
   - **Changed text:** card 14 Intent, "and `takeoverBranch.land` before each push of a takeover entry on a one-repository prime", and the row `TakeoverEntryPushOfThePrimeIsHeld`.
   - **The case:** `land` commits the entry through `b.commitEntry` before it pushes. On every push failure that is not a rejection it returns `errors.Join(..., b.dropEntry(rec))` (`internal/fabricengine/takeoverwrite.go`, `land`'s loop), so a failed takeover leaves no entry behind.
   - When the guard holds the push, the proposal returns an owed `operator_unpushed` `*PushFailedError`. It never says whether `dropEntry` runs.
   - If it does not run, the prime keeps an unpushed takeover entry claiming a run that no other machine sees, and the row only asserts origin is unmoved.
   - The plan should state the hold's disposition, either dropping the entry as `land`'s other failures do or keeping it as owed, and the row should assert it.

2. **MEDIUM. Card 9's marker-commit rule (the F7 proposal) leaves out "fabric merge commits (`MergeTrailerKey`)" by the trailer alone.**
   - **Changed text:** card 9 step 1, "leaving out fabric merge commits (`MergeTrailerKey`), whose marker step 3 settled".
   - **The case:** a single-parent commit on the source whose message carries a `Fabric-Merge:` trailer and which edits `.lym-anchor` is left out of the judged set. If it is the source's only marker commit, the merge passes, which is exactly the smuggled marker change D5 refuses.
   - Card 2 defines `IsFabricMergeCommit` as true only for a two-parent commit carrying the trailer (`02-records-paths-and-commit-predicates.md:29`). Card 14 relies on that two-parent bound: "A commit message alone admits nothing".
   - Card 9 should name `IsFabricMergeCommit` (and list it under Uses) rather than the bare trailer.

3. **MEDIUM. The F13 proposal states as intended a two-repository wait that can deadlock a run.**
   - **Changed text:** `00-overview.md`, "except the transition seam, which waits on a blocking git operation in both layouts (card 11), so no transition is passed with its records uncommitted", and the same sentence in card 11's Intent ("a two-repository run now waits at a weft merge where it skipped").
   - **The case:** in a two-repository hub, a fabric merge parked on a conflict (Finalize's conflict session works under it) leaves weft merge state. Card 10's `MergeStateOperation` blocks on any weft merge state there.
   - A status transition that fires while the run itself holds that merge open, before it concludes it, now sleeps until the merge ends. The merge ends only when the run moves on, so the run cannot move on.
   - The skip existed for this: `internal/loomcli/wiring.go:68-71` says "a mid-merge state skips the step", and `wiring.go:106` says "a transition whose commit was skipped mid-merge is caught up by the next one".
   - The proposal turns this into a stated, reasoned guarantee with no check that no transition fires during a run-owned parked merge.
   - Either scope the wait to one repository, or state why no transition fires while the run's own merge is parked.

**Proposals checked with no breakage found:** F2 (inverted field; two-repository and older records read settled), F3, F4, F8, F9, F10, F11, F12, F14 and F15.

**Tool calls**
- Read card 2's predicate definitions and card 6's anchor-commit text, to check how card 9's merge-commit exclusion and the add-at-anchor commit's trailer are defined. My context did not hold either card.
- Read `internal/loomcli/wiring.go:62-75` and grepped the landingshed conflict-session sites, to confirm why the seam skips mid-merge and that a conflict session runs under a parked merge. My context had only grep fragments of these.
